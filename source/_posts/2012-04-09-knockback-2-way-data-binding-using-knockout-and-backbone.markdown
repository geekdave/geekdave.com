---
comments: true
date: 2012-04-09 16:47:03
layout: post
slug: knockback-2-way-data-binding-using-knockout-and-backbone
title: 'Knockback: 2-way Data-Binding using Knockout and Backbone'
wordpress_id: 79
categories:
- Backbone
- Data-Binding
---

Knockout provides fantastic two-way data-binding; Backbone offers solid and scalable structure. Choosing between apples and oranges sucks, and thanks to Knockback, you don't have to!

<!-- more -->


## Knockout, Backbone, Knockback, Backknock... Wait, what?


[Knockback](http://kmalakoff.github.com/knockback/) is a fantastic little "bridge" library that brings together the MVP structure of **Back**bone and the magical data-binding goodness of [**Knock**out](http://knockoutjs.com/).  The library doesn't replace either one, but just smoothes over the rough edges where they stomp on each other, allowing you to create a Knockout ViewModel from a plain old Backbone.Model.

If you're not already familiar with Knockout, I strongly recommend taking 10 minutes to play around with the [excellent interactive tutorials](http://learn.knockoutjs.com/) on the project website.   They're fun, easy-to-follow, and will make the rest of this post much easier to understand.  Once you've seen two-way data-binding in action, you'll wonder why you ever wrote so much tedious boilerplate code to keep your UI in sync with your model.


## Show me the code!


The examples on the Knockback project page are a bit heavy and complex, which is unfortunate because the basic usage is quite simple.  Let's try out a super simple example:

{% gist 2344757 form.html %}

Here, we've set up a form using plain old Knockout data-bind'ing syntax.  The "value" and "checked" properties point to property keys ("name", "age", and "evil")  which we expect to find on our backing model.  The "data-bind" syntax means that we expect two things to magically happen:



	
  1. Automatically update the view whenever the model changes (i.e. when new data is retrieved from the server)

	
  2. Automatically update the model whenever the view changes (i.e. when the user enters a new value or toggles a checkbox)


To wire this up, let's set up a Backbone model to store our data, create a "bridge" view model using Knockback, and finally invoke the data-binding using Knockout.

{% gist 2344757 script.js %}

That's it!  Note the difference between the "kb" and "ko" API calls, which refer to Knockback and Knockout, respectively.  The call to "kb.viewModel" creates a Knockout view model which knows how to talk to a Backbone Model.  The call to "ko.applyBindings" tells Knockout to wire up the 2-way bindings with the UI.

Note: We specify "read_only" as "false" on the call to kb.viewModel, since we want the binding to be two-way.  That is, we expect changes to come from user interaction as well as programmatic updates.  If you don't want the UI to be editable by the user, and only want to react to changes that occur programmatically, you can leave this property as "true" (the default) which will save some memory and CPU cycles by only setting up a one-way binding.


## See it in action!


I created a JSFiddle to show a live demo of the above example.  [Check it out here](http://jsfiddle.net/geekdave/bjUBq/).  In this example, you can also click the "Change Model" button to update the backing Backbone.Model with new data.  Using the magic of KB/KO, you'll see the UI components update themselves automatically when the new data arrives.


## So What?


Sure, it would be possible to do all of this yourself, by writing the code to listen for a specific model property change, and then manually updating the UI.  But this kind of architecture quickly winds up as a tangled mess of spaghetti jQuery selectors.  Letting these libraries handle the task automatically means you can write less boilerplate code, which means fewer chances to introduce bugs.

The beauty of Knockback is that it's the only code that needs to know about both Knockout and Backbone.  Your other Backbone code can continue to interact with your Backbone Model just the same.  And once the view model is created using Knockback, it will walk and talk just like a vanilla Knockout view model.  So you can use the full Knockout API for setting up complex data-binding in your view, and it won't care that the data is ultimately originating from a Backbone Model.


## Models vs. View Models


In Knockout's case, the model plays a special role called a "view model" which is part of the [MVVM design pattern](http://knockoutjs.com/documentation/observables.html#mvvm_and_view_models).  In simple apps, the Knockout "View Model" and Backbone "Model" might seem kind of redundant.  Are they really just two different models that are storing the same data?  The key differences are:

View Model



	
  * Tightly-coupled bindings with particular DOM elements

	
  * May represent complete backing data, or a subset of it

	
  * Tied to UI using Knockout

	
  * Tied to Backbone model using Knockback

	
  * In simple apps, may have the same properties as the Backbone Model

	
  * Complex apps may have multiple Knockout view models per Backbone Model


Model

	
  * No knowledge of the DOM

	
  * Represents complete backing data

	
  * Updated directly from business logic / server sync operations

	
  * Tied to View Model using Knockback


In a simple form-based page, your View Model and Model will probably have the same properties.  But in a complex view with multiple tabs, grids, and forms, with a large and complex Backbone Model, you might choose to create a separate View Model for each sub-view on your page.  This way, each sub-view can have its own data bindings that are lazy-loaded as needed.  For example, you could lazy load a sub-view when you first click on a tab, and only then set up the viewModel and data bindings.  Since 2-way data bindings use memory and CPU time, it's best to be frugal when heavily using them.
