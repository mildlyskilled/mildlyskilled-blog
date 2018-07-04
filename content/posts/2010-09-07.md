---
title: "Mod_Rewrite and Zend Framework 1.10.x and Ubuntu"
date: 2010-09-07T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["Zend", "PHP", "linux", "ubuntu"]
type: "post"
---



So all the tutorials on configuring your development server to run Zend Framework successfully say that you need to enable mod_rewrite in apache2 so that .htaccess and Zend_Router* works well.

What they all say is to make sure it's loaded and enabled. I will raise my hand and say it wasn't that easy for me to find because Ubuntu lays out Apache2 a little differently from say Fedora. First of all in the /etc/apache2 folder there are a few folders to take note of.

sites-enabled and mods-enabled actually tell you what has been enabled or not and not an uncommented list in some httpd.conf file. Basically the folders in there have symbolic links that come from sites-available and mods-available. This perhaps is a better idea as you can add new mods and not have to recompile apache or whatever.

I digress, to enable mod_rewrite in apache2 in Ubuntu 10.04.1 you need to create a symbolic link from mods-available to mods-enabled to do this you can either use the command:


    sudo ln -s /etc/mods-available/rewrite.load /etc/mods-enabled/rewrite.load


or if you prefer GUIs you have to run nautilus as root so you can try


    sudo nautilus


and navigate to "/etc/mods-available", right-click on rewrite.load and select "make link" and the cut the new file created called "link to rewrite.load" paste it in mods-enabled and rename it to "rewrite.load "

Now all you need to do is restart your web server and you're all set to reroute anyhow you want.


    sudo /etc/init.d/apache2 restart
