---
title: "A note on Zend Pagination"
date: 2010-09-06T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["Zend", "PHP"]
type: "post"
---


Having gone round in so many circles I finally found out that when useing Zend Pagination and the current route you are using has more than one dynamic variable i.e


    $router->addRoute(
                    'listingswithpage',
                    new Zend_Controller_Router_Route(
                            '/listing/:type/:page',
                            array('controller' => 'listing',
                                'action' => 'index')
                    )
            );


Your pagination view helper gets a bit thrown if you go by the documentation. The first thing you need to do is find a way to specify the first page initially. I went with specifying my first page in my route definition so it looked something like this


    $router->addRoute(
                    'listingswithpage',
                    new Zend_Controller_Router_Route(
                            '/listing/:type/:page',
                            array('controller' => 'listing',
                                'action' => 'index',
                                'page' => 1)
                    )
            );


And that worked. Failing that you have to figure out a way to pass the complete URL path to you pagination view which might be something like "controls.phtml" or "pagination.phtml". and constructing the url properly there.

I will perhaps post a more detailed description of what this means but leave a comment if you are impatient and I will try and respond when I get a moment.
