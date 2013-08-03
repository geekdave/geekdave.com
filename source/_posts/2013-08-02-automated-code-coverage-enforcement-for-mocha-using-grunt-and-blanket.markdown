---
layout: post
title: "Automated Code Coverage Enforcement for Mocha using Grunt and Blanket"
date: 2013-08-02 09:04
comments: true
categories: 
---

*I wrote a similar post about integrating QUnit with Grunt and Blanket.  This is the Mocha version.*

There are several solutions for generating code coverage reports with Mocha, but most don't integrate well
 with Grunt, lack an "enforcement" option, or are overly complex.  Here's a quick and easy solution that 
 will get you up and running without the headaches.  

<!-- more -->

{% img right http://i.imgur.com/1ukQzJo.jpg [The Enforcer] %}

We front-end developers are spoiled for choice when it comes to unit testing frameworks, and their companion libraries.  At the end of the day, your choice of framework matters less than your team's dedication to writing meaningful, high-quality tests.  But how do you ensure that your production code is actually being tested?  As teams grow larger, and deadlines loom, it's often the tests that suffer.  How many times have you heard, "I didn't have time to write the tests, but I'll do them later..."  Sometimes, a cold-hearted, emotionless "enforcer" is the only solution.

At my company, we initially tried doing this on the honor system--we integrated BlanketJS into our Mocha runner, and voila! Developers could see code coverage statistics for their files right in their Mocha report:

{% img left /images/blanketreport.png [BlanketJS Report [BlanketJS Report]] %}

We asked developers to ensure their code had a certain minimum percentage code coverage before checking in.  But again, we still wound up with low coverage and the same excuses ("not enough time", "plan to do them later", etc).  Now, I'll be the first to admit that *I spouted out these excuses myself, just as much as the next guy.*  It was becoming clear that we didn't need a guideline... we needed an *enforcer*.

We had already begun using [Grunt](http://gruntjs.org) as part of our [Jenkins](http://jenkins-ci.org)/[Gerrit](http://code.google.com/p/gerrit/) continuous integration process.  Using the JSHint and grunt-mocha plugins, we were able to ensure that commits could not be pushed to the master branch if there were any lint errors, or unit test failures.  The next step was to ensure a minimum coverage threshold was reached *per-file* before a checkin was allowed.  

There was, at the time, no Grunt plugin available that allowed the same in-browser coverage report to be run headlessly using PhantomJS on the command line.  So in the spirit of open-source, I rolled up my sleeves and wrote one myself!

Enter [Grunt-Blanket-Mocha](https://github.com/ModelN/grunt-blanket-mocha).  

Other plugins look similar, but are different in that they:

* Only test *server-side* code
* Create *new instrumented copies* of your source code for coverage detection
* Generate coverage reports in HTML or JSON formats requiring a separate step to parse and evaluate coverage
* Do not *enforce* coverage thresholds, but just report on it

This plugin, however:

* Runs *client-side* mocha specs
* Performs code coverage "live" using BlanketJS, without creating separate instrumented copies
* Reports coverage info directly to the Grunt task
* Will fail the build if minimum coverage thresholds are not defined

This plugin is based on [kmiyashiro/grunt-mocha](https://github.com/kmiyashiro/grunt-mocha) and supports all the 
configurations of that plugin.  Please see that repo for more options on configuration.

Here's how to set it up.  Note that these instructions assume prior knowledge of Mocha and BlanketJS.  If you haven't yet gotten those two set up and working together, the rest of this post probably won't make much sense.

If you haven't used [Grunt](http://gruntjs.com/) before, be sure to check out the [Getting Started](http://gruntjs.com/getting-started) guide, as it explains how to create a [Gruntfile](http://gruntjs.com/sample-gruntfile) as well as install and use Grunt plugins. Once you're familiar with that process, you may install this plugin with this command:

```
npm install grunt-blanket-mocha --save-dev
```

Once the plugin has been installed, it may be enabled inside your Gruntfile with this line of JavaScript:

```js
grunt.loadNpmTasks('grunt-blanket-mocha');
```

In your project's Gruntfile, add a section named `blanket_mocha` to the data object passed into `grunt.initConfig()`.

Here's how it would look for a Mocha test runner called `test.html` and a minimum per-file threshold of 20%:

```js
grunt.initConfig({
  blanket_mocha: {
    all: [ 'specs/index.html' ],
    options: {
        threshold: 70
    }
  }
})
```

Use the `all` param to specify where your mocha browser spec HTML file lives.  
This works the same way as it does in the base `grunt-mocha` plugin.  

NOTE: Be sure to include the blanketJS script tag in your test html file

### Blanket Adapter

To allow Blanket to communicate with the parent Grunt process, add this snippet in your test HTML, after all the 
other scripts:

```html
<script>
    if (window.PHANTOMJS) {
        blanket.options("reporter", "../node_modules/grunt-blanket-mocha/support/grunt-reporter.js");            
    }
</script>
```

NOTE: The above path is assuming that the specs are being run from a directory one deeper than the root directory.  
Adjust the path accordingly.

NOTE 2: The conditional `if (window.PHANTOMJS)` statement is there because of the hacky way that messages are passed
between an HTML page and the PhantomJS process (using alerts).  Without this condition, you would get bombarded 
with alert messages in your in-browser mocha report.

### BlanketJS HTML Report

If you want to see blanketJS coverage reports in the browser as well (useful for visually scanning which lines have 
coverage and which do not) include this snippet it in your test html blanket and mocha.

```html
<script type="text/javascript" src="../node_modules/grunt-blanket-mocha/support/mocha-blanket.js"></script>
```

NOTE: The above path is assuming that the specs are being run from a directory one deeper than the root directory.  
Adjust the path accordingly.
Place this script snippet after your blanket.js script declaration.  This allows you to conditionally only enable this custom reporter if the `gruntReport` URL parameter is specified.  This way, you can share the same test runner html file between two use cases: running it in the browser and viewing the report inline, and running it via grunt. 

Finally, fire up grunt:

```
grunt
```

If all goes well, and all files in your project pass coverage, you should see something like this:

{% img /images/grunt-mocha/gruntMM.png [All files pass coverage] %}

Because every file that Blanket evaluated had code coverage equal to or greater than the threshold defined the Gruntfile, the build passed.

Now, let's say that we increase our coverage threshold to 99% and now we notice that the minimum threshold is not met:

{% img /images/grunt-mocha/gruntMM2.png [Some files fail coverage] %}

Now, the plugin outputs each file that failed, along with its coverage percentage, and fails the build.  If you have Grunt hooked up to your CI environment, such as Jenkins or Travis, this means that you can now block checkins that don't meet your code coverage criteria.

Note: If you want to see the code coverage percentages for *all files*, even those that passed coverage, use `grunt --verbose`.

I hope this is helpful!  If you give this plugin a try, please leave a comment and let me know how it goes for you...

You can find the Grunt-Blanket-Mocha plugin here: [https://github.com/ModelN/grunt-blanket-mocha](https://github.com/ModelN/grunt-blanket-mocha)