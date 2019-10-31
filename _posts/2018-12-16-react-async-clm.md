---
layout: post
title: "Async/Await in React Component LifeCycle Methods"
author: "Tyler Shin"
categories: frontend
tags: [React, React Hooks, Async]
image: asyncComponentDidMount.png
---
# Async/Awiat in React Component LifeCycleMethods

Recently I've thought of a purpose of the react methods. And one of my thought was 'Is it okay that Using asynchronous functions in the component lifecycle methods?'

This post  is for cleaning up my thoughts about the topic , and might not give you clear answer.

Let's start!

---

## Example Code

    import React from "react";
    import fetchComments from "./action";
    
    class BlogPost extends React.Component {
    	componentDidMount() {
    		const { post, dispatch } = this.props;
    
    		dispatch(fetchComments(post.id));
    	}
    
    	render() {
    		const { post, comments } = this.props;
    
    		<div>
    			<div>{post.title}</div>
    			<>
    				// ... some stuff related with the post information
    			</>
    			<div>Comment List</div>
    			<ul>
    				{comments.map(comment => (<li key={comment.id}>{comment.title}</li>))}
    			</ul>
    		</div>
    	}
    }

If we have common example code and scenario, it'll be easy to understand.

Example React Component in above code is very simple. It just has two logic.

1. Render some information about a post and comments.
2. Fetch comments data just after the component being mounted.

Now let's make our story. Suppose our design team requested that the scroll bar be moved to the comment area after the comment was loaded. 

There are two options to deal with the request. (Let's skip ref and specific scroll bar handling logic)

    class BlogPost extends React.Component {
    // ...
    // You can use Callback or Promise or Pub-Sub pattern rather than async/await
    	async componentDidMount() {
    		const { post, dispatch } = this.props;
    
    		await dispatch(fetchComments(post.id));
    		goToCommentArea();
    	}
    // ...
    }

    class BlogPost extends React.Component {
    // ...
    	componentDidMount() {
    		const { post, dispatch } = this.props;
    
    		dispatch(fetchComments(post.id));
    	}
    
    	componentDidUpdate(prevProps) {
    		if (prevProps.comments !== this.props.comments) {
    			goToCommentArea();
    		}
    	}
    // ...
    }

Which one is better? it's up to your team's convention?

I think of that they both have pros and cons. Let's think about it.

---

`Async/Await pattern`

## Pros

- It's easy to handle continuous asynchronous functions. If we have other 5 series of the asnyc functions and use React's life cycle methods only, it'll be difficult to track them. Because the continuous logic are scattered.

## Cons

- It can cause the changes of 'state' or 'screen(DOM)' not in React's way. Suppose that we waited 3s in componentDidMount method, then scroll is moved to the comment area. It's easy to follow where the scroll moving event is fired NOW. However, if there are a lot of methods it'll be chaos soon.
- Component life cycle methods are synchronous method.(...is this one of cons?)

`React Life Cycle Pattern`

## Pros

- The logic is working with React's Life cycle. It means that you can think and follow the logic in REACT way. It's important to keep only one(as less as you can) application flow. I believe the 'simplicity' is one of the most important things for the good code base.
- Also, I feel that React team made [React Hooks](https://reactjs.org/docs/hooks-intro.html) for handling there situation more easily. Because with React Hooks, we can gather our scattered logics and even reuse the functions.

## Cons

- It may cause performance issue. As you can see, BlogPost component will check old comments and new comments every time it receives new props or states. If there are a lot of conditional statement or other logic, it might cause performance issue.
- As I mentioned above, It scatters asynchronous functions. it can be burden to navigate through several places when creating and debugging.

---

## My Opinion

React Life Cycle Pattern might be a little bit nicer.

Because I've learned that the simple application flow with less side effect is important. And also I'm looking forward to React Hooks will solve some flaw of the Life cycle methods.

But If there is the huge performance issue or you feel that your code is so messed up, you can choose Async/Await pattern too.

If you have better / other idea, just give me a comment.

Thanks.
