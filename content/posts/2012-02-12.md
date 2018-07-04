---
title: "A standalone java/jar application from Scala sources using Proguard - Part 1"
date: 2012-02-12T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["scala", "proguard"]
type: "post"
---

After about an hour of digging around and looking for a way to package a simulation I wrote in Scala for a coleague, I came across this process of doing it. I have reduced it down to a bash script which will be at the bottom of this post but I thought I should go through the steps a bit as the script makes certain assumptions.

The first assumption is that you have the necessary libraries and such to run it they are as follows:




  * Scala Library (That goes without saying if you're going to be writing Scala code) I have a custom path for mine as I want to manually sync my libraries, compilers and so on with the latest versions. I have extracted my copy to ~/Bin/Scala-2.9.1.final this will be the path my bash script will assume, you must change this to your path if it's different.


  * Proguard: I am using version 4.7, version 4.4 comes with ubuntu but, seeing as I like to keep it at the latest and I am impatient my proguard lives in ~/Bin/Proguard4.7


  * Java (This also goes without saying. You will probably need the JDK as well as the JRE so install them both) Proguard uses the java libraries extensively so you need that.


My simulation is in the following directory structure
engine

    | - src
    | -- | --  Engine.scala
    | - build-app.sh
    | - scala-config.pro

The engine->src->Engine.scala is my scala application with the object Engine defined as my entry point this is important for the MANIFEST.MF file the script will be creating

**Now the steps**




  * First we can create a little folder to hold the output of the scala compiler so we create  a folder called bin inside of the engine folder itself so that the folder structure now looks like this
engine

    | - src
    | -- | -- Engine.scala
    | - bin
    | - build-app.sh
    | - scala-config.pro

by running

    mkdir bin


then run the following while you're inside engine which from now on I will refer to as root


    scalac -sourcepath src -d bin src/Engine.scala


this will compile the Engine.scala code into java classes inside of the bin folder


  * This completes the scala bit, everything else following is run with Java so we need a MANIFEST.MF file to compile a jar


    echo "Main-Class: Engine" > MANIFEST.MF


this will do it for you. Bear in mind you are still at the root level

Now at this point we change directory to the bin folder where scalac has created our classes and compile our jar using the manifest we have just created so this should do it:


    cd bin
    jar -cfm ../engine-no-scala.jar ../MANIFEST.MF *.*


You can obviously do this a different way by staying in the root and making sure your path is correct, personally I found this easier.

This above code will create our engine called engine-no-scala.jar with the MANIFEST.MF file
bear in mind at this point if you ran java -jar engine-no-scala.jar it may fail if your scala code required the scala library to run
mine does so now I need to package this WITH the scala library into a complete self-contained java application
now earlier I did say that my scala home was in a custom location so we need to keep that in mind for the next section

**PROGUARD**


  * Not only does proguard include the required libraries and so on, it also obfuscates and discards any unused classes and packages this is very nifty for distribution as it cuts down by orders of magnitude the size of your application.When I did this without proguard my application was over 8 MB because the entire scala library was in my jar
after I finished with proguard I am looking at an 8 KB application, and I think I can even get it smaller if I optimise my code a little bit morebut that's for another time. Proguard however needs a configuration file to work properly in my case, I went to their site and downloaded a[ sample scala-config file](http://proguard.sourceforge.net/#manual/examples.html) (click on the Scala link) I just made the following changes to the vanilla one they provided


    -injars engine-no-scala.jar
    -injars /home/kwabena/Bin/scala-2.9.1.final/lib/scala-library.jar
    -outjars engine.jar
    -libraryjars <java.home>/lib/rt.jar

    -dontwarn scala.**
    -verbose


I just changed the path to my jar file and my scala library and I also turned on verbosity (I like to know what's going on) my build script now has the following lines


    cd ../
    echo "Adding Scala library, obfuscating..."
    #proguard
    proguard.sh @scala-config.pro


after that you can remove any side effects from the script with


    rm -rf bin MANIFEST.MF engine-no-scala.jar


bear in mind that we changed the our current workind directory to the root level
after a few minutes of compiling, obfuscating, optimisation and shrinking you should have a nice jar to run by itself on a computer with ONLY the JRE installed


The complete script follows you can just copy this, save it into a file called "build-app.sh" as it is in my folder structure change the permissions to execute with


    chmod +x build-app.sh


and run it


    #!/bin/bash

    echo "Creating required directories"
    mkdir bin

    echo "Compiling scala sources"
    scalac -sourcepath src -d bin src/Engine.scala

    echo "Creating Manifest"
    #build manifest
    echo "Main-Class: Engine" > MANIFEST.MF

    echo "Building java archive from scala classes"
    #create jar
    cd bin
    jar -cfm ../engine-no-scala.jar ../MANIFEST.MF *.*

    cd ../

    echo "Adding Scala library, obfuscating..."
    #proguard
    proguard.sh @scala-config.pro

    #remove fluff
    echo "Removing fluff..."
    rm -rf bin MANIFEST.MF engine-no-scala.jar


I hope you find it as useful as I would have should this have been available earlier.
