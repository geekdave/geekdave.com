---
layout: post
title: "Linking 'raw' Github files from JSFiddle"
date: 2013-06-19 14:15
comments: true
categories: 
- Tips
---
For anyone who's not familiar with [JSFiddle.net](http://jsfiddle.net), I highly recommend checking it out.  It's a great little online playground for testing out new ideas, providing examples of bugs, proof-of-concepts, etc.

<!-- more -->

Chris Coyier of the famous [css-tricks.com](http://css-tricks.com) site wrote a great piece called [Seriously, just make a JSFiddle](http://css-tricks.com/seriously-just-make-a-jsfiddle/) where he describes the site's usefulness in collaborating with other developers.

For anyone already familiar with it, you may have used a common pattern where you linked to 3rd party JS libraries on Github like so:

`https://raw.github.com/documentcloud/backbone/master/backbone.js`

When viewing any file on Github, there is a "Raw" button that will give you a direct link to the file.  This is a handy way to reference JS files from services like JSFiddle.

Github discourages this, since they want repo owners to use Github Pages to host specific versions of their files.  They discourage it by serving files from the "raw" domain with `Content-Type: text/plain` instead of `Content-type:application/javascript`.

This wasn't a problem until recently, when Google Chrome [implemented a security fix](https://code.google.com/p/chromium/issues/detail?id=180007) that prevents JavaScript from being executed if it has an incorrect Content-type.  This makes sense when you're viewing pages outside of your control.  But when it's you who's deciding what scripts to include, it's a hassle.

Unfortunately, this broke a lot of JSFiddles, resulting in this error:  

`Refused to execute script from 'https://raw.github.com/documentcloud/backbone/master/backbone.js' because its MIME type ('text/plain') is not executable, and strict MIME type checking is enabled.`

Luckily, someone came to the rescue and [created a service](http://rawgithub.com) that will proxy any file from raw.github.com and serve it with the correct MIME type.

All you have to do is remove the "dot" between "raw" and "github", so that your request goes to `rawgithub.com` instead of `raw.github.com`.

So the above URL would instead be:

`https://rawgithub.com/documentcloud/backbone/master/backbone.js`

Happy Fiddlin'!