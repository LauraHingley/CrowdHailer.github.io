---
layout: post
title: Organising client JavaScript with Rollup.js
description: Project setup with ES6 module bundles and testing using Jasmine.js and Karma.js
date: 2015-11-12 08:33:05
tags:
- ruby on rails
- clojure
- haml
tumblr_url: http://crowdhailer.tumblr.com/post/80766441344/makers-acadamy-wandering-in-the-wild
---
[Rollup.js](http://rollupjs.org) is a great way to add structure to a large client-side code base.

You might not be wanting to tackle all of EcmaScript(ES6), yet still want to separate your JavaScript code into modules.
Previously the best way to do this was Browserify with common.js modules.
Rollup.js is the logical next step as it replaces the common.js modules with ES6 modules.
It is also a component of Babel (a full ES6 transpiler) so there is an easy next step to get working with the rest of ES6.

I have recently introduced Rollup.js into my development workflow.
The simple command line interface means it can be used without the overhead of taskrunners like(gulp).
It is easy to integrate with existing tools for testing.
I use the Karma test runner to execute my test framework of choice, which is currently Jasmine.

This post walks through my setup of Jasmine, Karma and rollup.
Want to jump straight to the code?
[Visit this repository to see the code](https://github.com/CrowdHailer/rollup-karma-jasmine-example).

*I will assume that you already have node and npm installed, need help see [Installing Node.js and npm](https://docs.npmjs.com/getting-started/installing-node).*

### Setting up your node project

We first need to create a `package.json` file to identify our project as a node project.
This can be done by hand or by executing `npm init`, which walks through setup.
The only setup we need is to give the project a name, let's call it calculator.

Next is to fetch the development dependencies we will use.
Fetch all the dependencies by executing the following.

{% highlight sh %}
```sh
$ npm install --save-dev \
    rollup \
    jasmine \
    karma \
    karma-jasmine \
    karma-firefox-launcher
```
{% endhighlight %}

At this point make sure that you have added `node_modules` to your `.gitignore` file.
It is not a good idea to commit all these node files to version control-something that I have done on too many occasions.

### Setting up Karma.js

The Karma test runner also comes with a handy project initializer which will create a `karma.conf.js` file.
Run the karma initializer with the following options.

{% highlight sh %}
```sh
$  ./node_modules/.bin/karma init

Which testing framework do you want to use ?
Press tab to list possible options. Enter to move to the next question.
> jasmine

Do you want to use Require.js ?
This will add Require.js plugin.
Press tab to list possible options. Enter to move to the next question.
> no

Do you want to capture any browsers automatically ?
Press tab to list possible options. Enter empty string to move to the next question.
> Firefox
>

What is the location of your source and test files ?
You can use glob patterns, eg. "js/*.js" or "test/**/*Spec.js".
Enter empty string to move to the next question.
> test/bundle.js
11 11 2015 17:04:52.298:WARN [init]: There is no file matching this pattern.

>

Should any of the files included by the previous patterns be excluded ?
You can use glob patterns, eg. "**/*.swp".
Enter empty string to move to the next question.
>

Do you want Karma to watch all the files and run the tests on change ?
Press tab to list possible options.
> no


Config file generated at "/home/peter/Projects/Calculator/karma.conf.js".
```
{% endhighlight %}

#### Note

- We do not need Require.js as we will be replacing this with Rollup.js
- You can select any browser from the list, just make sure that you have installed the correct browser runner.
- Don't worry about the missing test file, we will add it shortly.

### Creating our first test

Let's create a single test to check out Karma setup.
We will create a main test file and add an example test.

{% highlight js %}
```js
// test/main.js

describe("A suite", function() {
  it("contains spec with an expectation", function() {
    expect(true).toBe(true);
  });
});
```
{% endhighlight %}

To bundle this file use Rollup with its default settings and store the output.
*At this point the output will be unchanged, this is because we are yet to import any modules*

{% highlight sh %}
```sh
$ ./node_modules/.bin/rollup test/main.js > test/bundle.js
```
{% endhighlight %}

Now our bundle is available we can run the test.

{% highlight sh %}
```sh
$ ./node_modules/.bin/karma start --singleRun
```
{% endhighlight %}

This will show one passing test.
As `test/bundle.js` is just a derived file it does not belong in version control.
Add a line for it in the `.gitignore`.

### Simple test execution

Typing all those commands to execute the tests would be painful.
We can make life much easier by declaring them as npm scripts.
First one change is needed in `karma.conf.js`, we will set singleRun configuration to be true.

{% highlight js %}
```js
// karma.conf.js

module.exports = function(config) {
  config.set({
    // Continuous Integration mode
    // if true, Karma captures browsers, runs the tests and exits
    singleRun: true,
  });
};
```
{% endhighlight %}

A lot of functionality can end up in npm scripts and it is easy to write magical single line scripts that no-one understands.
My advice is to write small contained scripts for bundling and running the tests. Then have a third script that coordinates the small component scripts.
Add these scripts into `package.json`.

{% highlight json %}
```json
// package.json
{
  "scripts": {
    "test": "npm run -s test:build && npm run -s test:run",
    "test:build": "rollup ./test/main.js > test/bundle.js",
    "test:run": "karma start"
  },
}
```
{% endhighlight %}

To run the project tests we can just execute the npm test script.

{% highlight sh %}
```sh
$ npm -s test
```
{% endhighlight %}

This single simple test command follows npm conventions and is easy to remember.
Having a trivial test command is important if test driving the development of a project.

### Testing some real code

At the moment we have Rollup.js setup but it is redundant as we have no modules.
It is time to test some real code for a calculator.
The first feature a calculator should have is the ability to add two numbers.
Let's adjust the test file so that it imports the calculator and add a meaningful test.

{% highlight js %}
```js
// test/main.js

// Import all exported functions from the calculator source file
// They will be available as part of the calculator namespace
import * as calculator from "../src/calculator";

describe("A Calculator", function() {
  it("should be able to add two numbers", function() {
    expect(calculator.add(1, 2)).toEqual(3);
  });
});
```
{% endhighlight %}

The tests should now be failing, there is no code after all.
There are several features a good calculator might need.
To keep features separate the calculator module will import modules to gain its utility.

{% highlight js %}
```js
// src/calculator.js

// Import add from the `calculator/add.js` file
import add from "./calculator/add";

// Export add as one of the exports for the calculator module
export { add };
```
{% endhighlight %}

{% highlight js %}
```js
// src/calculator/add.js

// export the function add as the default for this module
export default function add(a, b) { return a + b; }
```
{% endhighlight %}

With the tests passing again our impressive calculator is taking shape.

### Building a distribution

The final thing, now we have working calculator components, is to bundle the source into a single file.
This is so it can be consumed in all the browsers that are not yet supporting ES6 modules.

Using Rollup.js again we bundle the distribution and save it in `index.js`.

{% highlight sh %}
```sh
$ ./node_modules/.bin/rollup \
    --format iife \
    --name calculator \
    src/calculator.js >  index.js
```
{% endhighlight %}

We pass the format and name options so that our final bundle is available in the global namespace and can be used by other js code.

The final step is to add index.js to `.gitignore`.

### Conclusion

This has so far been a convenient way for me to add some structure to my projects.
It has been much simpler to setup than Browserify with Gulp or Grunt.
Check out the [code for the calculator project](https://github.com/CrowdHailer/rollup-karma-jasmine-example) the result of this walk through.
Tell me what you think about this approach, if you would use it or if anything is missing.
