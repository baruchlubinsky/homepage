+++
Categories = ["Development", "Opinion"]
Description = "How to choose and make the best use of a framework."
Tags = ["development", "rails", "ember"]
date = 2014-10-11T08:42:59Z
menu = "main"
title = "A discussion on design patterns from the perspective of Ember.js and Ruby on Rails."
Draft = true
+++

Modern software development is all about frameworks. The vast majority of projects can be built on top of a vast amount of pre-existing work. Who ever stops to think about the percentage of lines of code in their Rails project that they wrote themselves? These frameworks have made it spectacularly easy to produce very complex applications. Consider an iOS app, written in a weekend by a twelve year old, that goes on to be downloaded a million times. But they have also made us lazy and dependent. Not that a developer should reinvent the wheel, but lazy to learn how the frameworks are designed in order to understand their best usage, and limitations.  

Doing modern web (and occasional mobile app) development, I'm constantly working with MVC frameworks. [Model–view–controller (MVC) is a software architectural pattern for implementing user interfaces.](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) The key word in that definition is "pattern". MVC is not a set of rules, its a suggestion. You can implement the pattern without using the words "model", "view", "controller" and you can have classes with those names without implementing the pattern. I think that it is essential for a software developer to be aware of the [design pattern](http://en.wikipedia.org/wiki/Design_pattern) she is using.

Its easy to to get the impression that MVC is the only reasonable way to build a data driven user interface. Especially web sites. A quick look at this [comprehensive list](http://en.wikipedia.org/wiki/Comparison_of_web_application_frameworks#Comparison_of_features) of web frameworks reveals just how popular MVC is. And for good reason, it is an extremely effective way model user interactions with a website (or mobile app).

This article isn't about the suitability of MVC as a pattern for web frameworks. Rather I want to look a bit deeper at the features of MVC that make it so useful, and to discuss how these can applied outside of the pattern with the same advantages. MVC predates Rails by at least 11 years. It was conceptualised without any intention of being used for modern web applications. 

MVC is just one example of the [Observer Pattern](http://en.wikipedia.org/wiki/Observer_pattern). Which in turn is a way of following the [separation of concerns](http://en.wikipedia.org/wiki/Separation_of_concerns) principle that is a corner stone of object orientated programming philosophy. That is the concept that code should be organised in small components, each with a well defined purpose. And that these logical roles are confined within their module. In fact separation of concerns should be a guiding principle of any software development project; it is essential for creating maintainable code.

The observer pattern is approach for ensuring that your concerns are separated. Of course the different modules of a program need to interact with each other, while each is performing its intended role. The observer pattern accomplishes this communication by having objects, called the subject, maintain a list of dependants that are its observers. When the state changes in the object, it notifies its observes and they can take the appropriate action. No data flows from the observer to the subject. In this way the responsibilities of each object (module, script, whatever) and their relationships to each other are clearly defined. 

MVC is easily explained in that context. A program is made up of components which are either a "model", "view" or "controller". A controller observes a model, and view observes the controller. The model is represents the underlying data and the user interacts with the view. Each of these components takes on a different appearance depending on the application. 

In a Rails application the model classes are responsible for wrapping the database; the controllers are responsible for retrieving and decorating the data; and the views are responsible for presenting the the data to the user. The web page is not the view. The class which parses the templates is the view (of MVC, obviously its convenient to call the template a view). The view does not communicate with the controller. When you make a request, any changes that were made on the data are sent to the server, which updates the model, and then a new MVC stack is instantiated to return the result.

Ember and its cousins are often referred to as frontend MVC frameworks. I don't have any experience with the others, but Ember is not an MVC. 

