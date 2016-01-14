# Writing code and tests in the same file - A javascript study

**TL;DR:**

This article explains why and how to achieve a simpler testing workflow (assuming you use [`babel`](https://babeljs.io/)):

* write code and tests in the same file
* in dev mode: compile and run everything in one go
* for production: ditch all the test code at compile time


## Motivation

_Tests..._ most of us want them, some write them but everybody hates setting them up.

It's a truth, setting up a proper test environment ultimately ends up in _config mayhem_ and frustration, when not completely discarded.

Once setup is done the usual workflow is:

1. write some code, `something.js`
1. write `something.test.js` next to it or in the `test` folder
1. compile / bundle code + test
1. reload the browser / node code
1. test runner restarts as well
1. wait for app crash or test report
1. GOTO 1

I've been doing this for long enough to notice a couple issues.

*First,* the code and tests are in separated files,  sometimes very far away from each other. Except for TDD adepts, we start writing `somecode.js`, then we create `somecode.test.js`, import the test framework and helpers, import the tested code...

> aint_nobody_got_time_for_that.gif

Oh and then we need to have the two files open, switch from one to the other. While not unbearable, the process is not really inciting.

Also, if we ever find an existing `stuff.js` file without `stuff.spec.js`, chances are that `stuff.spec.js` is never going to happen. Ever.

*Second,* you need a test runner. Even the most automated ones (like `ava` or `testem`) will require custom work to integrate them with your modern _"`babel` + `webpack`"_ bundling toolchain.

Then come the linter, `css` compiler, and you want it all to run in `watch` mode... Every one of these toolchain element is simple on its own but finding the right combination and configuration for the whole thing to work is a another thing. The javascript ecosystem ongoing quest for tooling simplicity is no surprise.

I found out that the test runner is usually the most problematic piece (experience may vary of course...).

Wouldn't it be awesome to just get rid of this step?

## There is another way

While learning from [Dan Abramov demoing (and live re-implementing!) redux in a jsbin](https://egghead.io/lessons/javascript-redux-writing-a-counter-reducer-with-tests), I loved how he simply writes the tests just below the code.

> screenshot

As jsbin re-executes the all code all the time, the tests are simply run, and potentially break, all the time as well.

Oh but we can't ship apps that includes and run all the tests, that's silly.

In the development environment however, we don't care if the app is slower or if the bundle is 20Mo. Chances are you're dealing with sourcemaps, un-minified, un-gzipped code and devtools slowing everything already.

This simpler workflow would be:

1. write code and tests in `foobar.js`
1. bundle the app
1. reload in browser / restart node
1. crash with report if tests fail
1. GOTO 1

When shipping to production, _simply_ discard the test code at compile time. _Oh wait._ **That** is the tricky part.

## Yes, we can!

## Caveats

## How `babel-plugin-discard-module-references` works
