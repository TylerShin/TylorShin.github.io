---
layout: post
title: "Journey for the best React structure #1 Directory Structure"
author: "Tyler Shin"
categories: frontend
tags: [react, redux, structure, folder structure, directory structure, component structure]
image: directory-structure.jpg
---

## Table of contents

0. [Introduce]({% 2019-10-31-react-structure.md %}) [ X ]
1. [Directory Structure]({% post_url 2019-10-31-directory-structure %}) [ X ]
2. [Components] ({% post_url 2019-10-31-component-architecture %}) [ X ]
3. Redux + Redux Starter Kit(RSK) [ ]
4. Reducer [ ]
5. StyleSheet [ ]
6. SVG [ ]
7. Build & Deployment [ ]
8. SSR [ ]  

---

# Directory Structure

---  

Organizing a directory structure is important because it affects the way we think. Especially it affects how you treat components and Redux stores. And it influences the complexity and redundancy of the entire app. So, think carefully and communicate with colleagues!
```
/src
  /atoms # atom components
    /button
    /typography
  /components # components
    /PostList
    /PostDropdown
  /pages
    /paperShow
    /signUp
  /hooks
  /reducers
  /actions
  /helpers
  /typings
    /api
```

### /atoms

This directory is for `atomic components` which is the smallest component. If your project's `atomic components` are entirely depends on external library(such as Bootstrap), this directory can be omitted.

### /components

This directory is for the common component which knows about our business logic.

### /pages

This directory is for the `page components`.

### /hooks

Custom hooks

### /actions

Redux actions

### /reducers

Redux reducers & slices. Redux reducers should be functional, data-driven, feature-based. Making reducer following by Page or Component is an anti-pattern. I'll explain it later.

### /helpers

helper functions

### /api

API requests and data models

### /typings

Custom type definitions

