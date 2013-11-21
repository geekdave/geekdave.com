---
layout: post
title: "Automated Code Coverage Enforcement for QUnit using Grunt and Blanket"
date: 2013-07-20 09:50
comments: true
categories: 
- Testing 
- QUnit
- Grunt
- Code Coverage
- Continuous Integration
---
BlanketJS is a fantastic code coverage companion for QUnit.  But until now, the coverage reports were limited to in-browser.  Here's how to automate the process using Grunt, allowing code coverage enforcement to be part of your continuous integration system.

<!-- more -->

{% img right http://i.imgur.com/1ukQzJo.jpg [The Enforcer] %}

We front-end developers are spoiled for choice when it comes to unit testing frameworks, and their companion libraries.  At the end of the day, your choice of framework matters less than your team's dedication to writing meaningful, high-quality tests.  But how do you ensure that your production code is actually being tested?  As teams grow larger, and deadlines loom, it's often the tests that suffer.  How many times have you heard, "I didn't have time to write the tests, but I'll do them later..."  Sometimes, a cold-hearted, emotionless "enforcer" is the only solution.

At my company, we initially tried doing this on the honor system--we integrated BlanketJS into our QUnit runner, and voila! Developers could see code coverage statistics for their files right in their QUnit report:

{% img left /images/blanketreport.png [BlanketJS Report [BlanketJS Report]] %}

We asked developers to ensure their code had a certain minimum percentage code coverage before checking in.  But again, we still wound up with low coverage and the same excuses ("not enough time", "plan to do them later", etc).  Now, I'll be the first to admit that *I spouted out these excuses myself, just as much as the next guy.*  It was becoming clear that we didn't need a guideline... we needed an *enforcer*.

We had already begun using [Grunt](http://gruntjs.org) as part of our [Jenkins](http://jenkins-ci.org)/[Gerrit](http://code.google.com/p/gerrit/) continuous integration process.  Using the JSHint and QUnit plugins, we were able to ensure that commits could not be pushed to the master branch if there were any lint errors, or unit test failures.  The next step was to ensure a minimum coverage threshold was reached *per-file* before a checkin was allowed.  

There was, at the time, no Grunt plugin available that allowed the same in-browser coverage report to be run headlessly using PhantomJS on the command line.  So in the spirit of open-source, I rolled up my sleeves and wrote one myself!

Enter [Grunt-Blanket-QUnit](https://github.com/ModelN/grunt-blanket-qunit).  This plugin extends the functionality of [grunt-contrib-qunit] which runs your QUnit tests headlessly using PhantomJS.  But additionally, this plugin reports Blanket code coverage statistics and will fail the build if the minimum thresholds you define are not met.

Here's how to set it up.  Note that these instructions assume prior knowledge of QUnit and BlanketJS.  If you haven't yet gotten those two set up and working together, the rest of this post probably won't make much sense.

If you haven't used [Grunt](http://gruntjs.com/) before, be sure to check out the [Getting Started](http://gruntjs.com/getting-started) guide, as it explains how to create a [Gruntfile](http://gruntjs.com/sample-gruntfile) as well as install and use Grunt plugins. Once you're familiar with that process, you may install this plugin with this command:

```
npm install grunt-blanket-qunit --save-dev
```

Once the plugin has been installed, it may be enabled inside your Gruntfile with this line of JavaScript:

```js
grunt.loadNpmTasks('grunt-blanket-qunit');
```

In your project's Gruntfile, add a section named `blanket_qunit` to the data object passed into `grunt.initConfig()`.

Here's how it would look for a QUnit test runner called `test.html` and a minimum per-file threshold of 20%:

```js
grunt.initConfig({
  blanket_qunit: {
    all: {
      options: {
		urls: ['test.html?coverage=true&gruntReport'],
        threshold: 20
      }
    }
  }
})
```

The `urls` param works the same way as it does in the base `grunt-contrib-qunit` plugin, with two important considerations.  First, you should pass `&coverage=true` as a URL parameter as shown above to trigger blanket.js.  This is the same as clicking the `Enable Coverage` checkbox in the QUnit report.  Second, you may pass another parameter such as `&gruntReport` to trigger the custom blanket reporter that talks to the grunt task.  See the next section for more info.

Be sure to register the task along with any other tasks you want run as part of the default Grunt task, like so:

```js
grunt.registerTask('default', ['jshint', 'blanket_qunit']);
```

(in the above example, I am also registering the jshint plugin which checks the code for errors)

### Custom Reporter

In order for Blanket.js to communicate with the Grunt task, you must enable a custom blanket reporter.  See the `grunt-reporter.js` file in the `reporter` directory in this repo.

You can enable this reporter in one of two ways:

1. As an inline proprety in your blanket.js script declaration, like so:

```html
<script type="text/javascript" src="blanket.js"
        data-cover-reporter="reporter/grunt-reporter.js"></script>
```

This method is suitable if your test runner html file is only used for headless testing.  Do not use this if you will be using this test runner html file in a browser, as it will spew a bunch of alerts at you (see the reporter implementation for the ugly `alert` hack used to communicate with phantomjs).

2. As a conditional option evaulauted at runtime in your test runner html, like so:

```js
<script>
    if (location.href.match(/(\?|&)gruntReport($|&|=)/)) {
        blanket.options("reporter", "reporter/grunt-reporter.js");
    }
</script>
``` 

Place this script snippet after your blanket.js script declaration.  This allows you to conditionally only enable this custom reporter if the `gruntReport` URL parameter is specified.  This way, you can share the same test runner html file between two use cases: running it in the browser and viewing the report inline, and running it via grunt. 

Finally, fire up grunt:

```
grunt
```

If all goes well, and all files in your project pass coverage, you should see something like this:

{% img /images/gruntgreen.png [All files pass coverage] %}

Because every file that Blanket evaluated had code coverage equal to or greater than the threshold defined the Gruntfile, the build passed.

Now, let's say that we increase our coverage threshold to 40% and now we notice some files that don't meet the criteria:

{% img /images/gruntred.png [Some files fail coverage] %}

Now, the plugin outputs each file that failed, along with its coverage percentage, and fails the build.  If you have Grunt hooked up to your CI environment, such as Jenkins or Travis, this means that you can now block checkins that don't meet your code coverage criteria.

Note: If you want to see the code coverage percentages for *all files*, even those that passed coverage, use `grunt --verbose`.

I hope this is helpful!  If you give this plugin a try, please leave a comment and let me know how it goes for you...

You can find the Grunt-Blanket-QUnit plugin here: [https://github.com/ModelN/grunt-blanket-qunit](https://github.com/ModelN/grunt-blanket-qunit)

*EDIT 19 September 2013: Fixed issue in sample Grunt config file where `all` options map was missing, causing the task not to run.  Thanks to Md. Ziaul Haq and Thomas Junghans for pointing this out.*  