---
layout: post
title: "The journey for the best React structure practice #1"
author: "Tyler Shin"
categories: frontend
tags: [react, redux, structure, folder structure, directory structure, component structure]
image: module-architecture.jpg
---

## DISCLAIMER

This is not an official architecture and not the best practice for everyone.
Please read this article for an one of the references.

Also, it has strong coupling with Redux and Typescript. So, if you're using Apollo client or MobX for state manager, the content might not meet with your taste.

## Background

I've been working with React for 3 years, and agonize over using React or Redux more. it's kind of retrospective for my past results.

I always welcome feedbacks.

## Purpose

- High component reusability (≈ less coupling)
- Productivity
- Secure system

---

## Table of contents

1. [Directory Structure]({% post_url 2019-10-31-react-structure %}) [ X ]
2. Component [ X ]
3. Redux + Redux Starter Kit(RSK) [ ]
4. Reducer [ ]
5. StyleSheet [ ]
6. SVG [ ]
7. Build & Deployment [ ]
8. SSR [ ]  


---

# Directory Structure

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

