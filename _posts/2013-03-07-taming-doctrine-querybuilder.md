---
layout: post
category : php
tagline: "Wrap up doctrine querybuilder"
tags : [doctrine, querybuilder, architecture, tutorial, refactoring, orm]
comments : false
excerpt: "Whenever I started a new project and used the Doctrine ORM in the past, I started to write methods in the Entity repository classes to get all the necessary data for the logic and the views."
---
{% include JB/setup %}

Whenever I started a new project and used the Doctrine ORM in the past, I started to write methods in the Entity repository classes to get all the necessary data for the logic and the views.

At first it seemed simple when I wrote a findAll function. But as the project grew, a ton of parameters were added to the same method to accomodate different usecases.

[Read more](http://drafts.easybib.com/post/44139111915/taiming-repository-classes-in-doctrine-with-the) @ drafts.easybib.com
