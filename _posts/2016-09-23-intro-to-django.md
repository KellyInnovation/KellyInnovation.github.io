---
layout: post
title: 'Introduction to Django: Our First Week'
---

This week, we started Django.  I am mostly impressed with its power.  It is surprising how much is built into this framework and easily accessible.

Starting the project with many directories is a matter of a few commands:
	mkvirtualenv project_name
	pip install django==1.8.14 (or preferred version)
	django-admin startproject project_name
	cd project_name
	./manage.py migrate
	./manage.py startapp first_app_name

That is it.  With those few commands, Django creates a main project directory, with a project sub-directory of the same name.  The main project directory comes preloaded with an __init__.py, admin.py, models.py, tests.py, and views.py.  There is also a manage.py for the whole project.  Many of these files are preloaded with appropriate information (settings.py has base directory, installed apps (with many preinstalled), a secret key (which we assign to one stored on our local environment), templates, and many more.

Creating the new app, with ./manage.py startup app_name similarly, adds many needed files.  The apps are the heart of the project.  They define models, templates, and views.  They define what makes the website function, how various aspects are connected, and how the pages look.

The inherent interconnectedness also allows for very concise language.  Each file is a matter of a few lines.  My only trouble with the concise language is ensuring I add connections in all needed places to make everything run cohesively.  These connections include urls within an app, url to connect app to project, adding apps to the settings file and importing where needed.

My other concern is how best to use tests to check functionality along the way.  Overall, I appreciate the work done by Django.