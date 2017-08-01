---
title: "Image Manipulation with Zend 1.x"
date: 2011-04-08T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["Zend", "PHP"]
---


I had been on the prowl for an MVC approach to image manipulation. Especially a method that did it the Zend way. In the forums and documentation for the framework the [developers didn't see a real need to implement](http://framework.zend.com/wiki/display/ZFPROP/Zend_Image+Proposal+-+Davey+Shafik) a "Zend_Image" class in the libraries as this was already available through the PHP GD and Imagick hooks.

Some of us however did and there are a [few tutorials](http://www.ajaxray.com/blog/2008/09/12/image-manipulation-in-zend-framework-with-php-thumbnailer-class-v20/) on how to incorporate the [PHPThumb library](http://www.gen-x-design.com/projects/php-thumbnailer-class/) into Zend. I have used the PHPThumb Library on the [mokocharlie.com](http://mokocharlie.com) project and found it I however stumbled on a much simpler way of doing this that feels more native to the framework Zend_Image is that implementation hidden [somewhere on the code.google.com](http://code.google.com/p/zend-image/) hosting servers.


    $image = new Zend_Image( APPLICATION_PATH.'/../public/media/images/large/'.$filename,
       new Zend_Image_Driver_Gd());
       $transform = new Zend_Image_Transform($image);
       if($image->getWidth() > $image->getHeight()){
           $transform->fitToWidth(295)
           ->save(APPLICATION_PATH.'/../public/media/images/medium/'.$filename);
           $transform->fitToWidth(147)
           ->save(APPLICATION_PATH.'/../public/media/images/small/'.$filename);
           $transform->center()->middle()->crop(74, 74)
           ->save(APPLICATION_PATH.'/../public/media/images/tiny/'.$filename);
       }else{
           $transform->fitToHeight(300)
           ->save(APPLICATION_PATH.'/../public/media/images/medium/'.$filename);
           $transform->fitToWidth(147)
           ->save(APPLICATION_PATH.'/../public/media/images/small/'.$filename);
           $transform->center()->middle()->crop(74, 74)
           ->save(APPLICATION_PATH.'/../public/media/images/tiny/'.$filename);
       }


The above is my implementation of it as I create three versions of an uploaded image.
