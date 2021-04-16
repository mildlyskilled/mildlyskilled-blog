---
title: "Jumping into Flutter (and Dart)"
date: 2018-07-30T15:44:10+01:00
tags: ["programming", "mobile", "mokocharlie"]
draft: false
type: "post"
author: "Kwabena Aning"
banner: "posts/2018-07-30/images/screen.png"
---

I tweeted a little while back that I had been looking at [Flutter](https://flutter.io) and [Dart](https://www.dartlang.org). This is not a tutorial on using flutter. Lots of people have done a much better job than I can ever hope to do here [This is a very good start](https://www.youtube.com/channel/UCFTM1FGjZSkoSPDZgtbp7hA) and a [plethora of resources](https://github.com/Solido/awesome-flutter). Here are my first thoughts...

## "What is this Flutter?"

I would describe flutter as a collection of tools that enables a developer to write some Dart code and have it compiled down to native code (ARM and x86) for the two main mobile platforms (android and iOS). The syntax of it looks a lot like Java although I do notice some Javascript-isms in there. Having said that coming from a strongly typed language background will help you a lot. I basically built my _use case_ app within a week of on and off development time.

## The Use Case

As a side project I have been working on [Mokocharlie](http://mokocharlie.com) for a very long time now. It's been through it's incarnations and at the moment it's a django app running on Heroku, with a couple of auxilliary services such as [cloudinary](https://cloudinary.com) dealing with our imaging needs. One look at the project should tell you that this a pivotal service to us. Having said that, that's not my core concern, I tend to leave the experts to do things I am not very good at, or things I don't have the time for.

Mokocharlie is an experience sharing platform, where we endeavour to share the best parts of (mostly) Ghana to the world. People take shots of the lifestyles of that country and post them on mokocharlie for the world to see. I have currently embraced the concept of single responsibility services and as such I have decided to build out APIs for the platform. The first being for photos. Of course photos will have other services it will talk to but i believed that the MVP would lie in pushing the photos out to the public really quickly and providing views into these experiences.

A _multi-headed_ approach to providing these experiences is a great use case for splitting out a JSON api for both desktop and mobile experiences and as such that's the approach I took. After building out the photos api in Scala with Akka HTTP, I proceeded to build the mobile experience for mokocharlie.

I am not going to go into detail on the implementation of the API but just to give a high level idea of how it's put together. The API is written in Scala with Akka HTTP, the database is on MySQl, and I use some JWT for 'stateless' authentication.

## What was the experience getting started

I have to say, I never really got stuck on anything. There are so many resources out there. From the getting started page to youtube videos on pretty much whatever you want to achieve with Flutter I was spoilt for choice on how to approach the solutions I had chosen (there's even a gitter channel for flutter).

## The Application

This was pretty much very easy to scaffold and proceed with as there are flutter commands for just about anything you need boiler plate for. The things that took the longest time to set up were my emulators. Once these were done I was able to get very quick feedback on what I was doing because flutter allows you to "hot reload" your code. The flutter framework itself consists almost solely of the concept of widgets and widget trees. This allows for nesting (sensible) UI elements, and also being able to traverse that tree for things like state management, and also for context. 

## Conclusions

Having used Flutter and Dart for a while there's a mind shift yuou need to have when writing Dart. I am not very keen on the language because of some nuances with the file system but it's a  compromise you need to make. And the return on that pain is in my opinion greater than the pain itself...
