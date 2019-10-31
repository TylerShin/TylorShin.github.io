---
layout: post
title: "Journey for the best React structure #2 Components"
author: "Tyler Shin"
categories: frontend
tags: [react, redux, structure, folder structure, directory structure, component structure]
image: button-component.png
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

# Components

I split components as 3 dimensions. The main purposes are enhancing a reusability and clarifying the responsibility for listening/dispatching redux store/actions.

## Atom component

![]({{ site.url }}/assets/img/button-component.png)  
Atom Component

This is the smallest component in our project. This is the smallest component of our project. These components are pursuing maximum reusability as can. There are some principals about Atom component.

1. Atom component **should not know about any business logic**.
    - It gets props as general as can. E.g. `{ text: string; }` is  okay, but `paperId: string;` isn't fit for it.
    - If it has coupled with business logic, it's very likely to use dispatch redux function or be restricted to certain situations. Eventually, it harms reusability.
2. Atom components are composable. `Atom + Atom = Atom | Component`
    - E.g. `<DropdownItem />` + `<DropdownItem />` could be `<DropdownList />`
3. Atom component doesn't take responsibility for positioning itself. In most cases, Atom component doesn't display enough information and it is just a source for composing the component. From my past experience, I realized that when Atom doesn't have positioning style rules, it's much more flexible.
    - When you use(consume) Atom component,  all style rules are possible and recommendable. However, declaring Atom with positioning style rule is an  anti-pattern.

        <Button style={{ marginLeft: 10 }} /> // Good
        
        const Button = ({ children ) ⇒ <button style={{ marginLeft: 10 }}>{children}</button> // Bad

    - But you can specify Atom's own default style as you want except positioning rules(like padding, margin, ...)
4. If your project highly relies on UI-library like Bootstrap, you can omit atom component. (But don't recommend)

## Component

![]({{ site.url }}/assets/img/paper-item-component.png)  
Component

This is normal component we usually know. Component knows about business logic and can connect with the Redux store. It means Component can dispatch Redux actions.

Component could be consists of either Atom component or Component. All variations are possible.

    Atom + Atom = Component
    Atom + Component = Component
    Component + Component = Component

We can imagine components are likely to become heavy and mess soon. So, it's important that divide component layer along with your project size. In my case, my colleague suggested me we'll better have `Complex Component` which is consists of heavy Component.

I think some people are curious about the definition of 'heavy'. Unfortunately, I have not found a proper definition. This is empirical, heuristic, and vary from project to project.

## Page Component

The page component has a direct connection with the Router. In most cases, it's the only child for `<Route />` and root component for the entire page. 

This component decide the page layout, position of each components.

It also has responsibility for the change of `location` or `route changes`.

However, It doesn't mean the Page component should take care of whole data fetching logic related to the location changes. Watching the change of location and notifying it to related Component is better architecture. it's more simple and easy to understand.

For example,

    import React, { FC, useEffect, useState } from 'react';
    import { useRouteMatch } from "react-router-dom";
    import fetchPost from 'api/post';
    import PostShow from 'components/postShow';
    import Layout from 'components/layout';
    
    // BAD
    const Page: FC = () => {
    const match = useRouteMatch("/posts/:postId");
    const postId = match.postId;
    const [post, setPost] = useState(null);
    
    	useEffect(() => {
    		fetchPost(postId)
    			.then(res => setPost(res));
    	}, [postId]);
    
    	return (
    		<Layout>
    			<PostShow post={post} />
    		</Laytout>
    	);
    };
    
    
    // GOOD
    // the target post is fetched at <PostShow /> not in <GoodPage />
    const GoodPage: FC = () => {
    	const match = useRouteMatch("/posts/:postId");
    	const postId = match.postId;
    
    	return (
    		<Layout>
    			<PostShow postId={postId} />
    		</Laytout>
    	);
    }