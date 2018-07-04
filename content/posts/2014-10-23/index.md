---
title: "Django Bug?"
date: 2014-10-23T14:15:42+01:00
draft: false
author: "Kwabena Aning"
tags: ["python", "django"]
type: "post"
---
After spending hours trying to debug why my settings file was not being found I stumbled across this

I've found the answer to my question.

If you've got an error in your settings, manage.py will swallow the exception and report as if the command does not exist.
This lead me down the path of incorrectly assuming my python path or venv environment was messed up.
If you want to diagnose this issue, run...

python app/manage.py help

... and it will show the exception. This, of course, was what was recommended by the django shell after it had told me that the command was not found.

This is clearly a bug in Django 1.4. It seems to me, an Exception should be reported regardless of what management command you run.

http://stackoverflow.com/questions/14885299/django-1-4-unknown-command-runserver
