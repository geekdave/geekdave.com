---
layout: post
title: "Introduction to Backbone Geppetto"
date: 2013-02-09 13:48
comments: true
published: false
categories: 
---
Building a large, multi-moudle application using BackboneJS?  Then Geppetto might be for you.  Geppetto lets you encapsulate your client-side business logic into Commands, de-coupling it from your models & views.<!-- more -->

# What are we talking about?

[Backbone.Geppetto]([http://modeln.github.com/backbone.geppetto/]) has been around for over a year now.  But developers are often confused about what problems Geppetto is intended to solve, and what types of projects would benefit from it.  This post is intended to answer those questions in detail.  

## In a Nutshell

{% img http://i.imgur.com/OgAHmqk.jpg 300 250 %}

Geppetto is basically three things:

1. **A Command Framework** -- Geppetto lets you extract client-side business logic (such as server calls, data processing, etc) into discrete and reusable components called `Commands`.  This allows you to share related functionality between different views and models, while keeping discrete pieces of business logic nicely encapsulated and highly-testable.
2. **A Modular Event Bus** -- Geppetto allows you to associate parts of your application (views, models & commands) to a shared `Context` object.  The Context is like air traffic control.  It allows related parts of your app to communicate via a common event bus, without bothering other parts of your app.  It's pub/sub, but instead of one global walkie-talkie, discrete parts of your app can communicate privately within their own context.
3. **A Shared Data Container** -- Geppetto provides a place for your shared client data to live.  Take the example of a model that's shared between multiple views.  Instead of that model living on a particular view--and all the other views needing to know about that view to access its model--the model can instead live on the shared Context.  

## Sounds good in theory; show me the code!

Ok, here's a quick example that shows all of Geppetto's pieces working together.   This is a slightly contrived example, where a simple Backbone View is meant to show details of a particular user account.  Clicking on the "Load" button in the View should fetch the latest data from the server, and populate the View.  If an error is encountered while loading the data, the View should communicate the error to the user.

**Note:** The examples in this post all use [RequireJS](http://requirejs.org) which is highly recommended (but not required) for use with Geppetto.  RequireJS further assists with de-coupling your app, by handling the tough job of managing inter-app dependencies.  

First, we define a Context object.  This is the central nervous system of the "User" screen in the application. 

``` js UserContext.js
define([
    "backbone",
    "geppetto",
    "src/js/commands/LoadUserCommand"
], function(
    Backbone,
    Geppetto,
    LoadUserCommand
) {
    return Geppetto.Context.extend({
        this.model = new Backbone.Model();
        this.mapCommand("loadUser", LoadUserCommand);
    });
});
```

Here we are creating a new Backbone Model and setting it as a property on the Context so that any component associated with the Context has access to it.  Next, we "map" a Command, so that when any component dispatches a "loadUser" event, the Context creates and executes a LoadUserCommand.  

Next, we define a View.  Every Context object needs to be associated with a particular instance of a View.  In the case of multiple views belonging to the same Context, this will be the outermost parent view.  

``` js View.js
define([
    "backbone", 
    "geppetto",
    "src/js/UserContext"
], function(
    Backbone, 
    Geppetto,
    UserContext
) {
    return Backbone.View.extend({
        initialize: function() {
            this.context = Geppetto.bindContext({
                context: UserContext,
                view: this
            });
            this.model = context.model;
            this.context.listen(this, "loadingError", handleLoadingError);
        }),
        render: function() {
            //...usual view rendering logic goes here...    
        },
        events: {
            "click #loadUserButton":  "handleLoadClick"
        },
        handleSaveClick: function() {
            this.context.dispatch("loadUser");
        },
        handleLoadingError: function(eventData) {
            alert("Error loading user: " + eventData.errorMessage);
        }
    });
});
```

Here we use the `bindContext` method to associate this view with an instance of our UserContext.  Since the Model is already defined on the Context, we simply set a reference to the Context's Model on the View.  This alleviates the View of needing to know (or care) how the Model is created.

Next, we set up a Backbone event listener to listen for a click event on the load button, and when this occurs, we dispatch a "loadUser" event onto the Context.  This way, the View can "ask" for something it needs, without needing to know about the underlying business logic for implementing the request.

The View also is responsible for communicating any errors to the user.  So we configure the View to listen for error events dispatched from the Context.  Again, note that the View does not know (or care) how these error events are created.  It just knows to listen for them, and display the error message passed via the event's payload object.

Finally, we define the LoadUserCommand which does all the heavy lifting of talking to the server and modifying the model.

``` js LoadUserCommand.js
define([
    "jquery"
], function ($) {
    var command = function(){};
    command.prototype.execute = function() {
        $.ajax({
          url: "load/user",
          success: handleUserLoaded,
          error: handleError
        });
    };
    command.prototype.handleUserSaved: function(data) {
        this.context.model = model.set(data);
        // do some other complex stuff with the new model data
    };
    command.prototype.handleError: function(xhr) {
        this.context.dispatch("loadingError", {errorMessage: xhr.responseText});
    }
}
} );
```
If the AJAX request is successful, the Command sets the response data on the Model.  Note that Geppetto automatically "injects" the Context into any Command that it executes.  So the Command conveniently has access to the Context and all its associated properties (such as the Model).  If the View is set up to listen for changes on the Model and re-render itself in response, then the cycle is now complete.

If the request encounters an error, the Command dispatches a "loadingError" event onto the context, passing the error message in the event payload.  Since the View is listening for this event on the shared Context, it can respond and display the error message.  Note that this error event case involves the Context acting as a simple event bus--the Context does not create a Command in response to this event--it just forwards the event on to the View that's listening for it.

This is a simple example that demonstrates Geppetto's pieces and responsibilities.  To review:

* **Backbone View**: Handles rendering data from the Model and responding to DOM events.  May also respond to Application events dispatched onto the Context, if those events should impact the display. 
* **Geppetto Command**: Encapsulates business logic such as talking to the server, and manipulating Model data.  Automatically invoked by Geppetto when an mapped event is dispatched on the Context.  Does not modify the view directly.
* **Geppetto Context**: Responsible for mapping Application Events to Commands, and instantiating new instances of those Commands when the appropriate event is dispatched.  Also acts as an Event Bus so that components can communicate indirectly via events.

# Philosophy

That all sounds really complicated.  Is it really worth it?  What value will it add to my project?

## Extreme Decoupling

Geppetto's pieces prescribe a level of decoupling that goes one step beyond most Backbone applications.  If your app is small, simple, and does not need to scale to multiple independent modules, then Geppetto might not be for you.  But if you're worried about complexity, testabilty, and reusability as your app grows & scales in the future, then Geppetto just might be the solution you're looking for.

## Did we really need another framework for that?

A staggering number of client-side JavaScript frameworks have been released in the past couple of years.  Many aren't happy about this, saying that it's fragmenting the JS development community, and making it hard for newcomers to know where to begin.  While this may be true, it's also a reassuring sign that the JS community has begun to take software *architecture* very seriously.

Since every project has its own unique architectural needs, there will naturally be some frameworks that will be a better fit than others.

While Backbone and Marionette are useful for apps of all shapes and sizes, Geppetto is intended mainly for large multi-module applications.  It adds several features that make it easier to write de-coupled, reusable, and highly-testable code. 

{% img http://i.imgur.com/idxB7ir.jpg %}

{% blockquote %}
I could do that with plain-old JavaScript...
{% endblockquote %}

Some are [quick to point out](http://vanilla-js.com/) that client-side frameworks add bloat, over-complicate design, and don't really do anything that couldn't be done with plain old JavaScript.  For small projects with relatively few developers, I am inclined to agree.  But ask yourself these questions:

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

## Turn your app into an Assembly Line

{% img left http://i.imgur.com/nmZTKWK.gif %} 
*Ford assembly line, 1913. ([Wikipedia](http://en.wikipedia.org/wiki/Assembly_line))*

I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line I am writing about an assembly line 


