---
comments: true
date: 2012-04-05 21:40:17
layout: post
slug: module-specific-subroutes-in-backbone
title: Module-Specific Subroutes in Backbone
wordpress_id: 13
categories:
- Backbone
- Routing
---
Backbone.Router provides an easy way to map complex URLs to application navigation and state.  But does it really scale for large modular apps?<!-- more -->

Like most of the Backbone examples you'll find on the web, the example Backbone Routes involve small/medium-sized apps.  These Routers read like a big "site map," with knowledge of the entire app's structure defined in a single Router configuration.  While this might seem manageable at first, it will quickly become a maintenance nightmare if you need to support a large multi-module application.


## It'll never get more complex than this, right?


To illustrate the slippery slope of monolithic configurations in a large multi-module application, let's visit the land of make-believe.  Let's pretend that I just started my own online bookstore, and I've set up some simple routes to handle what my site needs:

{% gist 2312084 smallrouter.js %}

We've got routes for the various pages in my app, as well as subroutes that accept parameters for navigating to a specific book's page based on its ID.  This simple list of routes fits my current needs just perfectly, and my site is small and basic so it's OK to have knowledge of all its functionality defined in one place, right?


## The bloat creeps in...


Flash forward 6 months... we've just been acquired by another company that also sells movies!  Well it turns out that searching for movies requires a totally different system than books.  Time to update the router!  We can't have one generic "search" route anymore, since searching for books and movies is different.  So instead, I'll create one subroute for "books" and one for "movies" and each can have their own "search" route.

{% gist 2312084 mediumrouter.js %}


## We've got technical debt, and the loan shark is calling...


Holy smokes!  We just received a gazillion dollars of venture capital, and now we're expanding into online games, streaming music, social networking, and lolcats!  Each one of these new products has their own development team which will be working around the clock to roll out new features.

Now we've got trouble.  As each app team implements new features, they add new routes.  Since we're using a single Router for the whole site, they're constantly resolving Source Control conflicts, and our once-tiny little router config is starting to read more like an epic novel...

{% gist 2312084 largerouter.js %}

We're only two weeks into new development, and our number of features (and necessary routes) is only going to grow with time.  This single router just isn't going to scale.  Wouldn't it be nice if each app team could have their own separate router config that's only concerned with their own module?

Given a URL of:

    
    http://example.org/module/foo/bar


...we need the notion of a main router that can handle directing to the proper "module" and then a series of module-specific sub routers that can handle the "foo" and "bar" parts in their own module-specific way.


> The secret to building large apps is NEVER build large apps. Break up your applications into small pieces. Then, assemble those testable, bite-sized pieces into your big application.




-- Justin Meyer, [Organizing a jQuery Application](http://jupiterjs.com/news/organizing-a-jquery-application)





## Let's make some bite-sized Routers!


To realize our goal, we're going to need a small extension called backbone.subroute.  Go to the [project page](https://github.com/ModelN/backbone.subroute), download backbone.subroute.js and include it in your project.  I created this project based on a [Gist by Tim Branyan](https://gist.github.com/1235317), with a few bug fixes and tweaks.  I'll be adding more documentation and tests as well.

First, let's re-define what our main router is.  Instead of representing a complete site map of the entire application, let's change our main router to only be responsible for redirecting to sub-modules.  Note the usage of the asterisk in the route definition.  This is called a [splat](http://documentcloud.github.com/backbone/#Router-routes), and is basically a wildcard, allowing the subroute to have any form.

{% gist 2312084 app.js %}


Basically our main router should only care about the first "directory" in the URL path.  After that, it's the module's job to define a subroute.  In the above example, we've set the sub-routes to be lazy-loaded and assigned to a namespaced application property the first time they are used.  On subsequent invocations, we'll just use the already-assigned namespaced property instead of creating a new subroute.







Here's what the Subroute for the "books" module looks like:






{% gist 2312084 books.js %}


 Note that we can define as many subroutes as we want, and the base router doesn't care.  Furthermore, these subroutes all use a _relative_ path so we don't have to redundantly specify the module name again.  This means that we can change the module name later on, or move it to another prefix later on, without having to change our routes.










Subroutes are very powerful, and allow our application to scale as new modules are added.  The main router will only need to be updated when new modules are added, and module developers are free to do as they please!







_**Updated 4/30/12: Fixed some incorrect sub-router initialization code where MyApp.Routers.Books was being set.  Thanks to [mminke](http://www.geekdave.com/?p=13#comment-10) for pointing this out!**_
