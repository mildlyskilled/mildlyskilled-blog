---
title: "Building an API for Mokocharlie's Community photos 2"
date: 2013-02-27T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["mokocharlie", "mobile", "api", "php"]
type: "post"
---

Last time out I introduced a basic idea for an API, this post will be about the basic architecture of the API framework. I have been listening to and reading on API best practices published by [APIGEE](http://offers.apigee.com/ebook-api-design-yt/, "Web API Design"). Although there are one or two things about this guide that I think pidgeon-hole me into certain standards, I agree in most part with a lot of what this e book has to say, I recommend you give it a browse at some point.

## Basic Architecture ##
Enough of this, on to the main point of this post. At the moment [Mokocharlie](http://mokocharlie.com "Mokocharlie") is built on top of the [Zend Framework](http://framework.zend.com "Zend Framework"). I put a lot of effort into that and I am not planning on rewriting it soon, I don't know maybe when we bump up versions to v5 (we are currently in v4). I will be building the API with [Laravel 4](http://four.laravel.com), although at the time of writing this it has not been officially released, I think there's enough stability in there to actually use it. So without further ado we proceed.

### Get Composer ###
If at this time you do not have composer installed on your computer please go [here](http://getcomposer.com, "Get composer") and install it now. I chose to do a global installation but you can do whatever you like as long as calling composer in your Laravel working directory works.

### Get Laravel 4 ######
* And set up your version control (optional but highly recommended)
Get the latest snapshot of the [Laravel 4](http://four.laravel.com "Laravel 4"), at this point you can set up a repository to manage your code. We use [unfuddle](http://www.unfuddle.com "Unfuddle") for all our mokocharlie projects and this will be no different.
I have however found that the best way to get laravel working as a moving target is track the [github repository](https://github.com/laravel/laravel).
* I create a branch on my local laravel installation called "laravel-tracker"

        git checkout -b laravel-tracker
* Add a remote and call it laravel with the url for the laravel git repo

        git remote add laravel git@github.com:laravel/laravel.git
* Before I start working on anything I make sure I am in master and then do a git pull on the develop branch from laravel

        git checkout laravel-tracker
        git pull laravel develop
* Do a composer update

        composer update
* Make sure the application is not broken

        php artisan serve
* If I am happy with what I see (in most cases I am), I just kill the server and then checkout master and continue working
