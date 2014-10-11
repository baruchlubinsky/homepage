+++
Categories = ["Development", "Opinion"]
Description = "How to choose and make the best use of a framework."
Tags = ["development", "rails", "ember"]
date = 2014-10-11T08:42:59Z
menu = "main"
title = "MVC is dead, long live MVC"
Draft = true
+++

Modern software development is all about frameworks. The vast majority of projects can be built on top of a vast amount of pre-existing work. Who ever stops to think about the percentage of lines of code in their Rails project that they wrote themselves? These frameworks have made it spectacularly easy to produce very complex applications. Consider an iOS app, written in a weekend by a twelve year old, that goes on to be downloaded a million times. But they have also made us lazy and dependent. Not that a developer should reinvent the wheel, but lazy to learn how the frameworks are designed in order to understand their best usage, and limitations.  

Doing modern web (and occasional mobile app) development, I'm constantly working with MVC frameworks. [Model–view–controller (MVC) is a software architectural pattern for implementing user interfaces.](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) The key word in that definition is "pattern". MVC is not a set of rules, its a suggestion. You can implement the pattern without using the words "model", "view", "controller" and you can have classes with those names without implementing the pattern. I think that it is essential for a software developer to be aware of the [design pattern](http://en.wikipedia.org/wiki/Design_pattern) she is using.

Its easy to to get the impression that MVC is the only reasonable way to build a data driven user interface. Especially web sites. A quick look at this [comprehensive list](http://en.wikipedia.org/wiki/Comparison_of_web_application_frameworks#Comparison_of_features) of web frameworks reveals just how popular MVC is. And for good reason, it is an extremely effective way model user interactions with a website (or mobile app).

This article isn't about the suitability of MVC as a pattern for web frameworks. Rather I want to look a bit deeper at the features of MVC that make it so useful, and to discuss how these can applied outside of the pattern with the same advantages. MVC predates Rails by at least 11 years. It was conceptualised without any intention of being used for modern web applications. 