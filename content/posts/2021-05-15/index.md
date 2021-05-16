---
title: "A Feed Reader: Authentication"
date: 2021-05-15T10:00:10+01:00
tags: ["programming", "rest", "engineering", "api"]
draft: false
author: "Kwabena Aning"
type: "post"
---

[Last time](/posts/2021-05-06/) out I wrote a little bit about starting my feed reader project. As I continue to ~~work~~ play with this project, I have come to that point where I am thinking about securing these endpoints a bit. We have to remember that this is going to be feeding a mobile application and as such, I can't really pin down the source requests... CORS might be a problem at this point.

Authentication however should not be a problem. A reader (that's what I will be calling my users) should be able to provide their email + password combination and get a JWT token that they use for subsequent requests.

Http4K gives us some nice tools to move us forward with this approach. The library provides filters, which allows us to do things with the request even before it hits the endpoint logic, and filters that can run after the request. In this case, I will be using a filter to listen for the _Authorization_ header, and parsing that into something that the business logic can use.

I have split this approach into a few parts.

- The filter and also the jwt/auth service.
- The authservice injects a reader repository, which finds a user with the provided credentials.
- The jwt service creates our token using the issuer, and secret.
- The token should then be passed into the header in subsequent request.
- The filter takes the Authorization header and verifies it with the JWT service, extracts a user Identifier and the route uses that downstream.

This is the application level security that we are going to try and accomplish this time around. I will not be talking about ssl and so on as that is a bit lower down the stack and that is not in the scope of this project at the moment. When it comes to deploying the application, we might have some thoughts on that.

A point of note... the database will store passwords encrypted using HMAC-SHA256, so when a user types in their password we will have to apply that algo, before we try to find the user from the database...

So with that squared out of the way, let's get into it.

Security Configuration
---

Let's modify our current tests to first of all include an authenticate endpoint and also to make sure that the previous tests that were passing without a token are now failing.

```kotlin
should("return UNAUTHORIZED from the user endpoint without a valid token"){
    app(Request(GET, "/reader/${reader?.id}")) shouldHaveStatus UNAUTHORIZED
}


// in the route
"/{id}" bind Method.GET to authFilter.then { request ->
    request.path("id")?.let { id ->
        val reader = runBlocking { readerService.readerById(id) }
        reader?.let {
            readerLens(it.toOutgoing(), Response(Status.OK))
        } ?: messageLens(Message("user not found"), Response(Status.NOT_FOUND))
    } ?: messageLens(Message("An ID is required"), Response(Status.BAD_REQUEST))
}
```

You now notice an Auth filter that checks for an authorisation header, extracts the token and makes sure that we have the correct token. Even before we get a token from requests we need to make sure we can issue tokens on authenticating the user. As usual the test first

```kotlin
should("authenticate given a user and a password") {
    val authRequest = Credentials(reader!!.email, newReaderRequest.password)
    val requestLens = Body.auto<Credentials>().toLens()

    val response = app(
        Request(POST, "/reader/authenticate")
            .body(requestLens(authRequest, Response(OK)).bodyString())
    )

    response shouldHaveStatus OK
    token = tokenLens(response).token
}
```

This authenticates the user and sets the token up in the test for subsequent test that will require testing with a valid token. The token is a JWT and we need these parameters as the minimum for generating and issuing tokens. [Auth0](https://auth0.com/learn/json-web-tokens/) has a great primer to what JWTs are and how they work. I think they do a better job at explaining it so I'm going to skip that here.

```yaml
jwt {
  issuer = "mildlyskilled.com"
  secret = "TO_BE_SET"
  secret = ${?JWT_SECRET}
  tokenDuration = 1d
  refreshTokenDuration = 7d
}
```

If you are familiar with HOCON style configuration you can see that my JWT_SECRET will be inject into this configuration at runtime from an environment variable. Using a project call [Config4K](https://github.com/config4k/config4k), I am able to extract config variables into data classes so with that said, this is what the JwtService looks like.

```kotlin
data class JwtConfig(val issuer: String, val secret: String )

interface JavaJWT {
    fun create(subject: String): Token?
    fun verify(token: String): String?
}

class JwtService(private val jwtConfig: JwtConfig) : JavaJWT {
    ...
}
```

I am leveraging the Auth0 Java-jwt library to take the work out of actually creating the JWTs etc. Now that we have a way of creating and verifying tokens, let's handle the process that actually creates a token when a reader provides some credentials.

```kotlin
import com.mildlyskilled.repository.ReaderRepository
import org.http4k.core.Credentials
import org.joda.time.Duration


data class Token(
    val token: String,
    val refresh: String?
)

class AuthService(private val readerRepository: ReaderRepository, private val jwtService: JwtService) {
    suspend fun authorise(credentials: Credentials): Token? =
        readerRepository.getReaderByCredentials(credentials.user, credentials.password)?.let {
            jwtService.create(it.id.toString(),  Duration.standardDays(1))
        }
}
```

This method returns a data class Token, I am not exposing the expiry and so on but I will have to if in future I want to put some logic in the front-end to decide when I need to do a refresh. Just a quick recap of what we're doing here.

- We get the input from the user
- Construct the Credentials instance from that... we can do validation at this point, but I'm not, I do however encrypt the password here.
- Pass the credentials object to the authorise method &#x2713;
- Authorise method goes into the database, finds the reader, and returns that reader data back to the JWT service to create the token for us. &#x2713;

How do we get the user credentials for this authentication bit? Through the authenticate endpoint in the Reader route. We extract credentials from the post body data and convert that to a Credentials object. This is what that looks like right now.

```kotlin
// the authenticate route
"/reader" {
    ...
    "/authenticate" bind Method.POST to { request ->
        val authRequest = credentialsLens(request)
        runBlocking { authService.authenticate(authRequest) }?.let {
            tokenLens(it, Response(Status.OK))
        } ?: messageLens(Message("Did not find a user with these credentials"), Response(Status.NOT_FOUND))

    },
    ...
}

```

I leave this as simple as possible. The AuthService just takes the creadentials from the lens and checks that we have a user with that email address. The password is encrypted and such so when we get that user with their email, we can use the SCrypt verify method to check the password once that passes we return a valid token with the JwtService.

Through a Filter
---

Now that we have a way to issue tokens, we should now build the functiality to validate those tokens. The filter is simple here extract the Authorization header which looks like 

```plain
Authorization: "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJkODE2ZTQxYS1jMDIwLTQ4YmUtYWVlMC05MGU4NzhlNGVjNzEiLCJpc3MiOiJtaWxkbHlza2lsbGVkLmNvbSIsImV4cCI6MTYyMTI2MDQ5Nn0.pPU1ve6uIaqFEq9Pxmss60fdGM6HrQhZ1F82Do9TBm4"
```

Once we have extracted the subject out of the token we pass it on to the next call filter in the chain... in this case the next in the chain is the route handler now.

```kotlin
package com.mildlyskilled.routes.filters

import com.mildlyskilled.model.outgoing.Message
import com.mildlyskilled.service.JwtService
import com.mildlyskilled.service.ReaderService
import kotlinx.coroutines.runBlocking
import org.http4k.core.Body
import org.http4k.core.Filter
import org.http4k.core.HttpHandler
import org.http4k.core.Request
import org.http4k.core.Response
import org.http4k.core.Status
import org.http4k.format.Jackson.auto

val Request.userId: String?
    get() = this.header("x-user-id")

class AuthFilter(private val jwtService: JwtService, private val readerService: ReaderService) : Filter {
    val messageLens = Body.auto<Message>().toLens()

    override fun invoke(next: HttpHandler): HttpHandler = { req ->
        req.header("Authorization")?.let { token ->
            jwtService.verify(token.split(" ")[1])?.let { userId ->
                runBlocking { readerService.readerById(userId) }?.let { reader ->
                    next(req.header("x-user-id", reader.id.toString()))
                }
            }
        } ?: messageLens(Message("Could not verify authorization header"), Response(Status.UNAUTHORIZED))
    }
}
```

That's all the filter does and now we can use that filter for any other routes we want to protect this will include all the feed routes.

- We get the input from the user &#x2713;
- Construct the Credentials instance from that &#x2713;
- Pass the credentials object to the authorise method &#x2713;
- Get a token &#x2713;
- Require a token for all protected routes &#x2713;

There are some other pieces that need to be in place for this backend including:

- Fetching individual news items
- Allowing us to mark a news item as read
- Allowing us to add new news channels/feeds

However I will not write about these because I think I have covered most of the important parts and also the code is freely available on my [github](https://github.com/mildlyskilled/reader). I will quietly put those bits in but my next post is going to be about building the user interfacing parts to this - we will be building a mobile application. Now I could use flutter and get the benefits of an iOS and an Android application with the same Dart codebase. I however want to challenge myself a bit and build the Android application separate from the iOS application and both applications sharing the same business logic using the Kotlin Multiplatform Mobile framework.
