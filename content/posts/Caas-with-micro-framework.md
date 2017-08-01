---
title: "Content as a Service with a Micro Framework"
date: 2015-06-30T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["scala", "scalatra", "content", "slick", "heroku"]
---
I had an idea of building a content management system with one of the Scala frameworks a while back and I decided on the Play Framework. In the middle of scoping things out in terms of my requirements, I decided that perhaps I did not want to build "yet another CMS." My initial idea was to build something like wordpress using Play, but the more I thought about it the more I questioned what exactly I wanted to achieve. And so I decided to build one (in my spare time) that answered some of these questions

## What exactly do I want to achieve
I feel that a content management system should not constrain you to what the developer thinks should be the way you write content. In that there's a WYSIWYG editor that allows you to embed images, there's a gallery "module" that allows you to upload images to a gallery and use that gallery when composing your content.
I imagine that the CMS should also allow you to impose some sort of hierarchy on your content so that you can nest pages within other, and allows you also to tag content for easy searches and that sort of thing.
I imagine that the CMS should also allow you to modify whatever themes that it ships with or allow you to create your own. All of these things are noble goals and I agree with them in principle. However I find that almost invariably, CMSs are hacked to achieve some custom goal that the CMS was not designed to achieve at the time it was being developed. Either that or it was clunky in its implementation.

So I was thinking, what if you remove the presentation layer entirely from the CMS so that the responsibility of displaying the content was with whoever was displaying it. They are responsible for composing and displaying their own content.

To that end I started looking at content as a service offerings and I realised that this has an emerging market and there are some decent services out there[^1]. I was at a conference last year or so and someone was talking about another service he was involved in using the play framework I believe[^2]. In any case, I was looking for a project to get my teeth into as I don't do as much coding as I used to. This is a good way to get going.
## What I am going to do
I feel like if this is going to be a JSON only service, I don't need any template rendering and any of those nice trimmings that come with Play Framework, session management and so on. I believe at some point I will need to have that for user management and permissions, but the core of this project is going to portable data exchange such as JSON. I suppose I could just use the Akka stack (Akka Http and persistent actors), but that would be slightly too Spartan at this point.

I want to deploy this very easily and I don't want to worry too much about where it's going. Heroku comes to mind for this one.

The framework should deal with routing in a nice readable way, and should not be too opinionated on how I organise my code. Doesn't come with too many template trimmings and I should just be able to get going with it quickly, I went for Scalatra.

Of course I don't want to care about the database server I am using I want to be able abstract that away in a sensible implementation, for this I am thinking about using Slick.

So here it is

* A CMS as a service
* Using a light, fast micro framework in Scala
* Slick for database work
* Heroku to deploy
* ~~Some documentation on the API itself using Swagger~~*

In the next post I will expand on the organisation of the content, and perhaps foray into some implementation details.


[^1]: https://www.contentful.com
[^2]: https://prismic.io/
[^working-with-JSON4s]: https://nosqlnocry.wordpress.com/2015/04/16/working-with-json-in-scala-using-the-json4s-library-part-one/
*Using swagger support with jackson json support breaks a lot of things
