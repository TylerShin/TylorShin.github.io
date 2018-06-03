---
layout: post
title: "How to choose Redux state vs React Component state?"
author: "Tylor Shin"
categories: frontend
tags: [frontend, react, redux]
image: reactUpdateHighlight.png
---

# Background
Redux is used for saving the states of the javascript app. For years, I've liked it pretty much.  
But I've experienced that sometimes it can be a burden for both application itself and developer too.  

Below are the frequent complains I've made.  

- Do I Really make and change at least 3 files to make dropdown is open or not?
- Should many components listen and compare the props to determine update self or not for changes in the comment input value? (The container component is more complex, the affected components are increased.)

Of course, you can minimize the 'component update' by making and connect reducers wisely. Though, I don't think it's easier than React's component state.  

I wanted to use React's component states for tiny component states to avoid complexity and getting performance enhancement.  

It works well for a while, but it became mess soon too.  
So I had to make some guideline or criteria for choosing a proper state handler.

# Criteria
- Should the state is saved at local storage or relatively permanent storage?
- Should the state is needed for server-side rendering? (ex: fetched data)
- Does the state affect whole app state or multiple components which aren't well related each other?

If a new state doesn't have any of them, I think of React component states will be the better idea.  

# Ref and More thoughts
- Dan Abramov [You might not need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)  
- Tyler Hoffman[React State vs. Redux State: When and Why?](https://spin.atomicobject.com/2017/06/07/react-state-vs-redux-state/)

React Context API will be game changer. However, I prefer to use Redux as I used before until now.
