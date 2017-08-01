---
title: "Mokocharlie Mobile Part 1"
date: 2017-07-31T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["android", "mobile", "react"]
---

After a massive break from hacking on Mokocharlie, I felt like I needed to get my mind of some other things and perhaps do something that I found to be fun on a weekend.

I have been meaning to build a ReSTful API for Moko for a while now, I experimented with PHP a long time ago. It was OK but I'm in a different space now so I decided to go with the Akka stack, and perhaps use Slick for the data layer. This will manifest itself later but for now this post is just to outline the checklist for my mobile app.

Things I need for a successful mobile application:

* A mobile API... this will need to be light-weight, and as we're working with some nice images, I will probably stick to the Cloudinary API to get me what I need for the image needs. This will have it's own post a little later but for now let us assume that there's an API we will make use of

* Mobile Development Environment - weapon selection: This has always been a thing for me because, I want the mobile application to work on the two main platforms. iOS and Android, I know Swift, and I can write decent Java but I don't want to write them both... so React Native... I think my Javascript is passable so I may be able to do it with React.
    * [Android Studio](https://developer.android.com/studio/install.html),
    * [Atom Editor](https://atom.io/),
    * `brew install Node`, and
    * `brew install Watchman`

These are the basics I need for this for the moment. Atom  will probably whinge a little at the absence of Flow, but you can install it or just ignore it altogether.
