---
title: "A Feed Reader"
date: 2021-05-10T16:00:10+01:00
tags: ["programming", "rest", "engineering", "api"]
draft: false
author: "Kwabena Aning"
type: "post"
---

I used to love Google Reader. I used it everyday to keep up to date with those tech sites that i subscribe to. I really liked how it worked... And then they shuttered the service and applications and suddenly I was out there looking for alternatives. Feedly is a good alternative, but this got me thinking, as I have been playing with mobile application development for a while, what if I built my own feed reader?

I have two options here:

- Build a mobile app that does everything - Upload an OPML file of my feeds to the the mobile app, have that parse and manage the state on the mobile app.
- Build a backend service, that takes the file upload and manages the state over there. And a thin mobile client that renders what my backend does.

It would certainly be quicker to go with the first option, but this is the kind of side project that allows me to learn some new frameworks, toolkits, libraries and such. And so true to tradition, I am going to use this opportunity to learn how to use, and build a backend service with [http4k](https://www.http4k.org/documentation/). The mobile bit I hope to try to build an experience with the [Kotlin Multiplatform Mobile toolkit](https://kotlinlang.org/docs/mobile/getting-started.html). This gives me a lot of breadth on what technologies I am going to expose myself to.

- http4K (including containerising the service, packaging, and maybe even native with GraalVM or Quarkus)
- KMM
- Android
- Swift UI

This post is going to start with my exploration with http4K, and building a simple ReST service with the following features.

- Authentication (JWT/OAuth...)
- Data upload(form my OPML file)
- And some additional features such as user profile management. If this project takes off I might not be the only one using it (I very much doubt that though, there are far more capable services out there with tenure such as Feedly).

Getting Started
---

I would like to preface this by saying that [this code is freely open](https://github.com/mildlyskilled/reader) to anyone who wants to follow this a bit closer. The code I present here are snippets of the complete body of work, and so might be out of date or just wrong as I have iterated over this for a few times. So the service should be able to:

- Expose some endpoints
- Receive uploads
- Parse xml
- And store the XML and nicely normalised data in a nice database

Without further ado let's get into it shall we? Firstly http4K is my choice for a vareity of reasons... paramount on that list is the spartan nature of the footprint out of the box... According to the documentation outside of the kotlin stdlib it has no other dependencies [^1] - I like that.

Also I like that it approaches its solution design from a functional perspective. Running a server as function, entities are immutable. I can't help but think (I might be wrong though) that it was inspired by the older [http4s](https://http4s.org/v1.0/) library in Scala.

The project maintainers have provided a CLI to help with _scaffolding_ a project with a "getting started" template. The cool kids use [sdkman](https://sdkman.io/) nowadays so I decided to jump on that train although there is a brew package for it as well if you are on a Mac OS.

After running the CLI the project was generated and I got on with getting it running for the first time. It is usually fine to just accept all the defaults suggested by the "wizard" but in my case because I will be (de)serializing XML I included Jackson XML support.

Testing
---

As with any good projects you start of with... we should begin with tests. It allows us to be clear on what the service should be doing (the contract) before we actually start writing code and so without further ado.

```kotlin

class ReaderTest : ShouldSpec() {
    override fun listeners(): List<TestListener> = listOf(ServiceTestListener)

    private val newReaderRequest = NewReaderRequest(
        firstName = "Joe",
        lastName = "Bloggs",
        email = "joe.bloggs@mildlyskilled.com",
        password = "test"
    )

    private var reader: MildlySkilledReader? = null

    init {
        "Reader Application" {
            should("register a new reader") {
                val requestLens = Body.auto<NewReaderRequest>().toLens()
                val readerLens = Body.auto<MildlySkilledReader>().toLens()
                val newReaderResponse = app(Request(POST, "/reader/new").body(requestLens(newReaderRequest, Response(ACCEPTED)).bodyString()))
                reader = readerLens(newReaderResponse)
                reader?.firstName shouldBe "Joe"
                reader?.lastName shouldBe "Bloggs"
                reader?.email shouldBe "joe.bloggs@mildlyskilled.com"
            }

            should("get OK from user endpoint") {
                app(Request(GET, "/reader/${reader?.id}")) shouldHaveStatus OK
                val message = Body.auto<MildlySkilledReader>().toLens()
                val reader = message(app(Request(GET, "/reader/${reader?.id}")))
                reader.firstName shouldBe "Joe"
                reader.lastName shouldBe "Bloggs"
                reader.email shouldBe "joe.bloggs@mildlyskilled.com"
            }

            should("return NOT_FOUND where we don't have a user") {
                val messageLens = Body.auto<Message>().toLens()
                app(Request(GET, "/reader/${UUID.randomUUID()}")) shouldHaveStatus NOT_FOUND
                app(Request(GET, "/reader/${UUID.randomUUID()}")) shouldHaveBody messageLens(Message("user not found"), Response(
                    NOT_FOUND)).bodyString()
            }

            should("get OK from the feed endpoint") {
                app(Request(GET, "/feed/${reader?.id}")) shouldHaveStatus OK
            }

            should("get not found if a non existent id is passed") {
                app(Request(GET, "/feed/${UUID.randomUUID()}")) shouldHaveStatus NOT_FOUND
            }

            should("accept uploaded feeds") {
                val sample = ReaderTest::class.java.getResource("/xml/my_rss_feeds.opml")?.readText()
                val base64 = Base64.getEncoder().encodeToString(sample?.toByteArray())
                val importRequest = ImportRequest(
                    readerId = reader!!.id.toString(),
                    payload = base64
                )

                val requestLens = Body.auto<ImportRequest>().toLens()
                app(Request(POST, "/feed/import").body(requestLens(importRequest, Response(ACCEPTED)).bodyString())) shouldHaveStatus ACCEPTED
            }
        }
    }
}
```

These two tests are basically asserting that give app (the Http Routing Handler) and the following requests, these are the responses we should get. Of course they will fail because we do not have the endpoints defined yet so...

```kotlin
// file com/mildlyskilled/route/Feed
package com.mildlyskilled.route
import org.http4k.core.Method
import org.http4k.core.Response
import org.http4k.core.Status
import org.http4k.routing.RoutingHttpHandler
import org.http4k.routing.bind

fun feed(): Array<RoutingHttpHandler> =
    arrayOf(
        "/feed" bind Method.GET to {
            Response(Status.OK).body("feed")
        }
    )
```

I include the imports here so it's clear which bits I am using for what. As you can see this is very basic and should make the first test pass

```kotlin
// file com/mildlyskilled/route/User
package com.mildlyskilled.route

import org.http4k.core.Method
import org.http4k.core.Response
import org.http4k.core.Status
import org.http4k.routing.RoutingHttpHandler
import org.http4k.routing.bind

fun user(): Array<RoutingHttpHandler>  =
    arrayOf(
        "/user" bind Method.GET to {
            Response(Status.OK).body("user")
        }
    )
```

How do we tie all this together? We bring it all into the main application entry point like so.

```kotlin
package com.mildlyskilled

import com.mildlyskilled.route.feed
import com.mildlyskilled.route.user
import org.http4k.core.then
import org.http4k.filter.DebuggingFilters.PrintResponse
import org.http4k.filter.DebuggingFilters.PrintRequest
import org.http4k.routing.routes
import org.http4k.server.SunHttp
import org.http4k.server.asServer

val app = routes(*(feed() + user()))

fun main() {
    val server = PrintRequest()
        .then(PrintResponse())
        .then(app)
        .asServer(SunHttp(9000)).start()

    println("Server started on ${server.port()}")
}

```

When you get around to running this you should be able to hit http://localhost/9000/(feed/user) and get something back and in addition to that you should see what your request looked like and what the response looked like because of the filters `PrintRequest()` and `PrintResponse()` respectively.

Some Parsing
---

So if you have ever seen an OPML file, it's pretty basic. Here's an extract of the OPML I exported from my google reader account before it got shuttered

```xml
<?xml version="1.0" encoding="UTF-8"?>
<opml version="1.0">
    <head>
        <title>Kwabena subscriptions in feedly Cloud</title>
    </head>
    <body>
        <outline text="News" title="News">
            <outline type="rss" text="The Guardian World News" title="The Guardian World News" xmlUrl="http://feeds.guardian.co.uk/theguardian/rss" htmlUrl="https://www.theguardian.com/uk"/>
            <outline type="rss" text="BBC" title="BBC" xmlUrl="http://news.bbc.co.uk/rss/newsonline_uk_edition/front_page/rss.xml" htmlUrl="https://www.bbc.co.uk/news/"/>
        </outline>
    </body>
</opml>

```

And here's how to represent it as Kotlin using the Jackson deserialiser.

```kotlin
package com.mildlyskilled.model


data class Opml(val head: Head, val body: Body)
data class Head(val title: String)
data class Body(val outline: List<Outline>)
data class Outline(
    val text: String,
    val title: String,
    val type: String?,
    val xmlUrl: String?,
    val htmlUrl: String?,
    val outline: List<Outline>?
)
```

A lot of magic happening here but this is the function to parse it. By using a [Lens](https://ncatlab.org/nlab/show/lens+%28in+computer+science%29) extractor we are able to get the data class out of a String. So it's perhaps a more palatable way of doing reflection.

```kotlin
package com.mildlyskilled.parser

import com.mildlyskilled.model.Opml
import org.http4k.core.Body
import org.http4k.core.Request
import org.http4k.core.Method
import org.http4k.format.JacksonXml.auto

object OpmlParser {
    fun parse(request: Request): Opml {
        val messageLens = Body.auto<Opml>().toLens()
        return messageLens(request)
    }
}
```

and of course the test. I will be adding more to this test, such as making sure we have a title and text and so on, but for now I think this should suffice

```kotlin
class OpmlParserTest {
    @Test
    fun `Should extract models from OPML Feed`() {
        val sample = OpmlParserTest::class.java.getResource("/xml/my_rss_feeds.opml")?.readText()
        val feed = OpmlParser.parse(Request(Method.GET, "/").body(sample!!))
        feed.body.outline.forEach {
            when (it.text) {
                "News" -> assertEquals(2, it.outline?.size, message(it.text))
                "Others" -> assertEquals(1, it.outline?.size, message(it.text))
                "technology" -> assertEquals(12, it.outline?.size, message(it.text))
                "Comics" -> assertEquals(1, it.outline?.size, message(it.text))
                "Funny" -> assertEquals(7, it.outline?.size, message(it.text))
                "OSS" -> assertEquals(5, it.outline?.size, message(it.text))
                "Ubuntu" -> assertEquals(2, it.outline?.size, message(it.text))
                "Geeky Stuff" -> assertEquals(8, it.outline?.size, message(it.text))
                "tech" -> assertEquals(3, it.outline?.size, message(it.text))
                "Thinkers" -> assertEquals(1, it.outline?.size, message(it.text))
            }
        }
    }

    private fun message(title: String) = "Wrong feed count for $title"
}
```

So what do we have now:

- Exposing endpoints &#x2713;
- Receive uploads
- Parse xml &#x2713;
- And store the XML and nicely normalised data in a nice database

Some persistence
---

Once again with all of this, we start with some tests... we need to prepare some tables and build our entities and then wrap those in some services (business logic). We are using the excellent [Jetbrains Exposed](https://github.com/JetBrains/Exposed) library here. One thing I will have to point out though is that I have not foind support for coroutines in http4K so I will have to make the request/response blocking. I will look into that later (if I ever need to optimise).

I decided to put in some connection pooling so I went with HikariCP, I have long been a fan of C3P0 but I feel HikariCP has more going for it at the moment. It works nicely with Exposed through my JdbcRepository here from whence all my other DbRepositories extend.

```kotlin
open class JdbcRepository(config: DbConfiguration) {
    private fun dataSource(config: DbConfiguration) = HikariDataSource(HikariConfig().apply {
        driverClassName = config.driver
        username = config.user
        password = config.password
        jdbcUrl = config.url
        maximumPoolSize = 3
        isAutoCommit = false
        transactionIsolation = "TRANSACTION_REPEATABLE_READ"
        validate()
    })

    init {
        Database.connect(dataSource(config))
    }
}
```

My repositories extend this class so we always have a connection in scope when invoking them.

```kotlin
// Tables
object FeedTable : UUIDTable("FEED") {
    val name = text("FEED_NAME")
    val title = text("FEED_TITLE")
    val type = varchar("FEED_TYPE", length = 5)
    val xmlUrl = text("XML_URL").nullable()
    val htmlUrl = text("HTML_URL").nullable()
    val icon = reference("ICON", IconTable).nullable()
}

// Entity
import com.mildlyskilled.repository.db.FeedTable
import org.jetbrains.exposed.dao.EntityClass
import org.jetbrains.exposed.dao.UUIDEntity
import org.jetbrains.exposed.dao.id.EntityID
import java.util.UUID

class Feed(id: EntityID<UUID>): UUIDEntity(id) {
    companion object : EntityClass<UUID, Feed>(FeedTable)
    var name by FeedTable.name
    var title by FeedTable.title
    var type by FeedTable.type
    var xmlUrl by FeedTable.xmlUrl
    var htmlUrl by FeedTable.htmlUrl
    var icon by FeedTable.icon

    fun toOutGoingNewsFeed() = with(this) {
        OutgoingFeed(
            name = this.name,
            title = this.title,
            type = this.type,
            xmlUrl = this.xmlUrl,
            htmlUrl = this.htmlUrl,
            icon = this.icon?.value
        )
    }
}

class Section(id: EntityID<UUID>): UUIDEntity(id) {
    companion object : EntityClass<UUID, Section>(SectionTable)
    var name by SectionTable.name
    var title by SectionTable.title
    var owner by SectionTable.reader
    var created by SectionTable.created
    var updated by SectionTable.updated
    var feeds by Feed via SectionFeedTable

    fun toOutgoingSection() =
        with(this){
            OutgoingSection(
                name = this.name,
                title = this.title,
                owner = this.owner.value,
                created = this.created.toString(),
                updated = this.updated?.toString(),
                feeds = this.feeds.map {
                    it.toOutGoingNewsFeed()
                }
            )
        }
}
```

You may have noticed that there is now this function in the entity what converts the actual entity to a new data class. This is imperative if you want to return this as serialised data, because if you do you will run into a STACK OVERFLOW error because of the many to many relationship defined on the feeds field in the Section entity. Also I am converting the datetime fields to string because our serialisers here do not know how to handle joda-time datetime fields and exposed uses those to model sql datetime so to get around this catch 22, I opted for the low-tech approach.

And then we can go ahead and implement some repositories and services like so:

```kotlin
// repository interface
interface FeedRepository {
    suspend fun getReaderSections(readerId: UUID): UserFeed?
    suspend fun persistFeed(readerId: UUID, opml: Opml): List<Unit?>
}

// Service for the feed
package com.mildlyskilled.service

import com.mildlyskilled.model.incoming.Opml
import com.mildlyskilled.repository.FeedRepository
import java.util.UUID

class FeedService(private val feedRepository: FeedRepository) {
    suspend fun readerFeed(readerId: String) =
        feedRepository.getReaderSections(UUID.fromString(readerId))

    suspend fun saveFeed(readerId: String, opml: Opml): Boolean =
        feedRepository.persistFeed(UUID.fromString(userId), opml).isNotEmpty()
}
```

Finally we can update our routes to look like what follows:

```kotlin
package com.mildlyskilled.route

import com.mildlyskilled.model.outgoing.Message
import com.mildlyskilled.model.outgoing.UserFeed
import com.mildlyskilled.service.FeedService
import kotlinx.coroutines.runBlocking
import org.http4k.core.Body
import org.http4k.core.Method
import org.http4k.core.Response
import org.http4k.core.Status
import org.http4k.format.Jackson.auto
import org.http4k.routing.RoutingHttpHandler
import org.http4k.routing.bind
import org.http4k.routing.path
import org.http4k.routing.routes

fun feed(feedService: FeedService): Array<RoutingHttpHandler> =
    arrayOf(
        "/feed" bind routes(
            "/{id}" bind Method.GET to { req ->
                val messageLens = Body.auto<Message>().toLens()
                req.path("id")?.let { userId ->
                    val responseLens = Body.auto<UserFeed>().toLens()
                    runBlocking { feedService.readerFeed(userId) }?.let { readerFeed ->
                        responseLens(readerFeed, Response(Status.OK))
                    } ?: messageLens(Message("This reader was not found"), Response(Status.NOT_FOUND))

                } ?: messageLens(Message("Provide a valid User ID"), Response(Status.BAD_REQUEST))

            },
            "/import" bind Method.POST to { request ->
                val importLens = Body.auto<ImportRequest>().toLens()
                val importRequest = importLens(request)
                try {
                    val import = String(Base64.getDecoder().decode(importRequest.payload))
                    val opml = Parser.parseOpml(Request(Method.GET, "/").body(import))
                    if (runBlocking { feedService.saveFeed(importRequest.readerId, opml) }) {
                        messageLens(Message("Successfully processed import"), Response(Status.ACCEPTED))
                    } else {
                        messageLens(Message("Could not parse and import feed"), Response(Status.BAD_REQUEST))
                    }
                } catch (e: IllegalArgumentException) {
                    messageLens(Message("Invalid payload"), Response(Status.BAD_REQUEST))
                }
            }
        )
    )
```

A point of note here, we have not built in any of the authentication/authorisation yet... That will come in subsequent posts but for the mean time this should suffice for a good jump off point.

So what do we have now:

- Exposing endpoints &#x2713;
- Receive uploads and persist &#x2713;
- Parse xml &#x2713;
- And store the XML and nicely normalised data in a nice database &#x2713;

Also a last bit on the testing... I swapped out the default testing libraries in the http4k installation for kotlintest because I wanted to use the should spec and I also prefer how it uses listeners for setUp and tearDown... I like to wipe the database before each test suite for a variety of reasons including avoiding race conditions.

So in a nutshell, this has been a pretty painless process so far and I am happy I have taken this on, because it gives new a new option to quickly build out lightweight backend services. Once again if you would like to look at the code that I produced for this it is all here at [https://github.com/mildlyskilled/reader](https://github.com/mildlyskilled/reader). I can see this being used as some end to end test harness, or even for a function as a service offering. In my next post I will write about how I am going about securing this reader service, how to retrieve news items, and perhaps jump into the scaffolding the mobile application.

[^1]: https://www.http4k.org/documentation/
