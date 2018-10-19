---
layout: post
title: "Journey for minimizing bundle.js"
author: "Tylor Shin"
categories: frontend
tags: [webpack, performance, webpack4, migration]
image: afterLodash.png
---

## Background  
I'm working at [Pluto network](https://pluto.netwrok) and making [Scinapse](https://scinapse.io) now.  

I've been living at S.Korea which has one of the best internet infrastructures.  
But I knew that 70% of our users are from Southeast Asia or Africa. and generally, they had poor network infrastructure.  

Pluto's vision statement is **Breaking down barriers in academia**.  

We all know a large JS size can undermine the user experience and become a barrier.  
As a one man in Pluto network, I wanted to break this barrier as long as possible.  

And even if the internet infrastructure is good, nobody likes the slow first screen or the slow first interaction.

There are many studies and papers about user experience concern with page speed.  

[Web Site Delays: How Tolerant are Users?](https://scinapse.io/papers/2740580754)   

[The impact of network service performance on customer satisfaction and loyalty: High-speed internet service case in Korea](https://scinapse.io/papers/2062570156)

---

## Tech-Background  
Scinapse is using [Universal Rendering](https://en.wikipedia.org/wiki/Isomorphic_JavaScript).  

Yeah. we all know it brings faster initial page load and SEO optimization.  
However, it never speeds up the initial interaction.  


What I mean is this.  
as soon as you connect to the Scinapse, the users can see the search input element. (from server side rendering)  
But if the user write something in input field before JavaScript is loaded, it will be erased because our javascript initialize it.  
All interactions should not be allowed until JavaScript is loaded.(toggle login modal, etc...)

I remember that Angular team has made "rewind" method, but IDK how it is now.  

---

## Define Problem
I already used Webpack's production mode and set NODE_ENV as production.  
Also applied [UgilfyWebpackPlugin](https://webpack.js.org/plugins/uglifyjs-webpack-plugin/) which decreases bundle size a lot.  

As the result, the bundle size was 1.8MB without Gzip.

To further reduce, I had to know where it took up a lot of capacity.  
Fortunately, Webpack team provides the nice plugin called [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer).  

![before minimize bundle]({{ site.url }}/assets/img/before-minimize.png "before minimize bundle")  

As you can see, Lodash, MomentJS, Material-UI, JQuery take so many spaces.

I planed to apply [Tree Shaking](https://webpack.js.org/guides/tree-shaking/) and remove unneeded dependencies and upgrade webpack along with loaders.  

---

## Apply Tree shaking  
The most important part is connecting Typescript with Babel.  
I solved this problem with following articles.  

- [Webpack official tree shaking doc](https://webpack.js.org/guides/tree-shaking/)
- [Blog post about Typescript Tree-Shaking](https://alexjoverm.github.io/2017/03/06/Tree-shaking-with-Webpack-2-TypeScript-and-Babel/)

The key is using ES2015 module system with Typescript and hand over it to Babel.  

After applying this, Lodash and some other libraries became small.

**RESULT**  

![after apply tree shaking]({{ site.url }}/assets/img/afterLodash.png "after apply tree shaking")  

**1.8mb -> 1.6mb**

---

## Replace MomentJS to date-fns

I strongly recommend to change MomentJS to other library.  
I know MomentJS has long history and strong community power.  
However, it's too heavy unless you use almost every feature of MomentJS.  

There are other options.  

I changed it to [dafe-fns](https://date-fns.org/) which is recommended by [Dan Abramov's tweet](https://twitter.com/dan_abramov/status/805030922785525760).  

It is much lighter and has fancy API.  

You can choose [dayjs](https://github.com/xx45/dayjs).  
It's super lightweight but I couldn't have the chance to use it yet.

**RESULT**  

1.6mb -> 1.53mb

---

## Apply tree shaking to material-ui
[official document](https://material-ui-next.com/guides/minimizing-bundle-size/)  
It's very easy to follow.
And I think it's not optional for who are using Material-UI.  

**RESULT**  

![after reduce material-ui]({{ site.url }}/assets/img/afterMaterialUI.png "after reduce material-ui")  

1.53mb -> 1.3mb

---

## Remove JQuery dependency
Changed the toast message library from Toastr to [Notie](https://github.com/jaredreich/notie) which has zero dependency.  


**RESULT**  

![after remove JQuery]({{ site.url }}/assets/img/afterRemoveJQuery.png "after remove JQuery")  

1.3mb -> 1.23mb

---

## Upgrade to Webpack4
I've done this in the following list order.  

1. upgrade all loaders version to latest.
2. upgrade typescript version to latest.
3. remove unneeded options from `webpack.config.js`
4. Add new optimization rules to `webpack.config.js`

the most important part is 4.  
this [blog post](https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a) was very helpful.

and you should use UgilfyWebpackPlugin for optimization.  

**RESULT**  

![after upgrade webpack]({{ site.url }}/assets/img/afterWebpack4.png "after upgrade webpack")  

1.23mb -> 1.14mb

---

## Conclusion
**BEFORE**

![before minimizing with gzip]({{ site.url }}/assets/img/beforeMinimizeGzip.png "before minimizing with gzip")  

**AFTER**

![after minimizing with gzip]({{ site.url }}/assets/img/afterMinimizeGzip.png "after minimizing with gzip")  

**GZipped Result**  
**495kb -> 291kb**(52% of the original)

It was fun to define the problems, and solve the problem one by one.  

--- 

## What's more?
### Add loading state before loading JavaScript
it depends on designer's choice and work yet.  

### Code splitting
it requires changes in the deploy process. And also increases code complexity a lot.  

![next time, baby](https://media.giphy.com/media/aowvYVUErM0SY/giphy.gif  "next time, baby")  

Thanks.  
Pluto breaks down barriers in academia.
