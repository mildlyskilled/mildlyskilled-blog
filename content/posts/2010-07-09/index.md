---
title: "Having multiple instances of Zend Navigation in a Single View Zend 1.x"
date: 2010-07-09T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["Zend", "PHP"]
type: "post"
---

so I have been playing around with the Zend Framework rather extensively over the past few weeks and it has been a very rewarding experience.

I am posting this here because it was a bit difficult finding a work around for the issue I describe below.

Zend Navigation had been giving me a lot of problems especially because I could not get the Navigation object passed to my view object to render two different navigation configs. The top menu which remains the same for all pages and is rendered in my layout and other sub menus rendered in my individual view pages.

Now getting to it:

in my bootstrap I initialise my navigation with something like this.

```   
    $config = new Zend_Config_Xml(APPLICATION_PATH . '/configs/navigation.xml', 'nav');
    $miscNavConfig = new Zend_Config_Xml(APPLICATION_PATH . '/configs/miscnavigation.xml', 'nav');
    $navigation = new Zend_Navigation($config);
    $miscNavigation = new Zend_Navigation($miscNavConfig);
    Zend_Registry::set(Zend_Navigation, $navigation);
    Zend_Registry::set('MiscNavigation', $miscNavigation);
    $view->navigation($navigation);
```




Now  lines 2, 4, and 6 are the new lines I have added for additional nagivation. I beleive you should be able to add new configurations as you move along if you need to.

The next bit was retrieving the sub navigation in my controller:





    $miscNavigation = Zend_Registry::get('MiscNavigation');
    $this->view->miscnavigation = $miscNavigation;





You can do this in your view but because I route several of my views through this controller I want to have to do it only once.

And finally in my view:





    $options = array('ulClass' => 'submenu');
    echo($this->navigation()->menu()->renderMenu($this->miscnavigation, $options));





Bear in mind that I use the renderMenu method instead of passing $this->miscnavigation into the menu method. If you do that you will overwrite any other instances of menu you have on the page with the miscnavigation object.
