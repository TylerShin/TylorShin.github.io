---
layout: post
title: "React Unit test with Jest, Typescript, Redux"
author: "Tylor Shin"
categories: workflow
tags: [jest, redux, react, test, unit test]
image: packageJson.png
---

# Unit Test

## Background
We've been used Jest with [Enzyme](https://github.com/airbnb/enzyme).  
However, because of the decorators(or HoC) we barely unit tests for the React components.  
To do proper test, I have to mock dependencies, and it's kind of annoying thing and even sometimes it's impossible.  

In the meantime, Jest Provided [Snapshot Testing](https://jestjs.io/docs/en/snapshot-testing) with [react-test-renderer](https://reactjs.org/docs/test-renderer.html).  
Thanks to these tools, I had no reason to use Enzyme and it makes component testing simple.

This document is the simple guide for setting test environment who're using TypeScript, React, Redux together.  


## Common setting
You should set the jest options in the package.json(You can set it with jest.config.js or else. If you want to write config to other file, follow [this guide](https://jestjs.io/docs/en/configuration.html).)  

![package.json setting]({{ site.github.url }}/assets/img/packageJson.png)

Above is my current config options. we will explore key configs.  

### transform  
This is pre-processing setting. I used [ts-jest](https://github.com/kulshekhar/ts-jest) to transpile Typescript to Javascript.  You can make your own pre-processing logic like below image.(we had used below pre-processing logic before.)  

![pre-processing setting]({{ site.github.url }}/assets/img/preProcessing.png)


### moduleNameMapper  
This option is needed to re-map module to another one. it's similar with module mocking.  
In above config, we are trying to map asset files(jpg, png, ...) with `fileMock.js`.  
And also map style files(css, scss, less) with [`identity-obj-proxy`](https://github.com/keyanzhang/identity-obj-proxy) package.  

Because TypeScript Compiler can't handle those files, we should mock them.

For example, it makes mock for below code.
```js
const styles = require("./paperItem.scss");
const OPEN_SORTING: require("./open-sorting.svg");
const ORCID_LOGO: "orcid-logo.png";
```

module mocking(avoiding unneeded module files) are rely on [ES6's Proxy feature](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy). If you want to know about the core, I recommend to visit and study about it.  

### setupFiles
the paths to modules that run some code to configure or set up the testing environment before each test.  
(From Jest official docs.)

![setupFiles]({{ site.github.url }}/assets/img/preload.png)

above settings are needed because jsDOM has [issue](https://github.com/geelen/react-snapshot/issues/93) with scrollTo method.


**Common JavaScript(TypeScript) test**
WIP

**Action Test**
WIP

**Reducer Test**
WIP

**Dumb Component Test**
WIP

**Container Component Test**
WIP

# E2E Test
**Dependencies**
- [nightwatch](http://nightwatchjs.org/)
