---
layout: post
title: "Introduction to Backbone Geppetto"
date: 2013-02-09 13:48
comments: true
categories: 
---
Building a large, multi-moudle application using BackboneJS?  Then Geppetto might be for you.  Geppetto lets you encapsulate your client-side business logic into Commands, de-coupling it from your models & views.<!-- more -->

{% img http://i.imgur.com/OgAHmqk.jpg 300 250 %}

## In a Nutshell

Geppetto is basically two things:

* **A Command Framework** -- Geppetto lets you extract client-side business logic (such as server calls, data processing, etc) into reusable commands.  This allows you to share related functionality between different views and models.
* **A Modular Event Bus** -- Geppetto allows you to associate parts of your application (views, models & commands) to a shared "Context" object.  The Context is like air traffic control.  It allows related parts of your app to communicate via a common event bus, without bothering other parts of your app.

Geppetto's pieces prescribe a level of decoupling that goes one step beyond most Backbone applications.  If your app is small, simple, and does not need to scale to multiple independent modules, then Geppetto might not be for you.  But if you're worried about complexity, testabilty, and reusability as your app grows & scales in the future, then Geppetto just might be the solution you're looking for.

## Ok, I'm intrigued... tell me more!

BackboneJS is well-suited for projects of all shapes and sizes, Backbone Geppetto is a plug-in for Backbone Marionette.  While Backbone and Marionette are useful for apps of all shapes and sizes, Geppetto is intended mainly for large multi-module applications.  It adds several features that make it easier to write de-coupled, reusable, and highly-testable code.  This will be the first post in a series of deep-dives on Geppetto.

{% img http://i.imgur.com/idxB7ir.jpg %}

{% blockquote %}
I could do that with plain-old JavaScript...
{% endblockquote %}

Haters are [quick to point out](http://vanilla-js.com/) that client-side frameworks add bloat, over-complicate design, and don't really do anything that couldn't be done with plain old JavaScript.  For small projects with relatively few developers, I am inclined to agree.  But ask yourself these questions:

1. How big will this project grow over time?
2. How many different developers will work on this project in the future?
3. How reusable will important parts of my code be? 
4. Will every developer working on this project be a JavaScript Ninja?

As the Single Page Web App becomes a viable solution for more Enterprise Software companies, these questions become increasingly relevant.  

Well-designed Enterprise Software:

* involves a reusable "platform" layer, that can scale to support an arbitrary number of modules
* allows new developers to join a project and ramp up quickly
* provides an easy way to "extend" and reuse common base functionality
* allows developers to focus on business-specific problems, and not architectural problems

Every JavaScript architect has their own personal favorites for accomplishing the above.  But a common set of frameworks and tools has emerged as being especially good for creating large-scale applications:

* RequireJS - Handles dependency management across large, multi-module codebases
* Backbone - Provides clean structure for managing views, events, and data
* Marionette - Reduces boilerplate for complex apps with nested views

When [my company](http://www.modeln.com) set out to build our next-generation UI, we settled on the above tools.  But something was still missing.  We enjoyed the Model/View separation provided by Backbone.  But we were missing the Controller piece.  Now, Backbone is [commonly regarded as an MV* framework](http://lostechies.com/derickbailey/2011/12/23/backbone-js-is-not-an-mvc-framework/)--not MVC--because it doesn't claim to have a Controller at all.  Most commonly, writing client-side business logic the "Backbone Way" involves embedding that logic in your View or your Model.  Again, for smaller and simpler apps, this works perfectly.  Backbone.Model's built in "sync()" method allows an easy 1:1 mapping between a Model and a REST URL.  But our needs were much more complex.  We required the ability to share client-side business logic between different views and different models.  So we needed a place for that business logic to live.  

{% img left http://i.imgur.com/nmZTKWK.gif %} 
*Ford assembly line, 1913. ([Wikipedia](http://en.wikipedia.org/wiki/Assembly_line))*

I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line 


A staggering number of client-side JavaScript frameworks have been released in the past couple of years.  Many aren't happy about this, saying that it's fragmenting the JS development community, and making it hard for newcomers to know where to begin.  While this may be true, it's also a reassuring sign that the JS community has begun to take software *architecture* very seriously.

JS was once a language for hobbyists, providing a way to add cute animations or simple client-side validations to a page.  But the innovations that have come along with these frameworks have proven JS to be a first-class language, worthy of large-scale, multi-module applications.