---
layout: post
title: "Differences between HTMLElement.innerText and HTMLElement.textElement "
author: "Tylor Shin"
categories: tech
tags: [tech, css]
image: cutting.jpg
---

Differences from innerText
Internet Explorer introduced node.innerText. The intention is similar but with the following differences:

While textContent gets the content of all elements, including script and style elements, innerText does not.
innerText is aware of style and will not return the text of hidden elements, whereas textContent will.
As innerText is aware of CSS styling, it will trigger a reflow, whereas textContent will not.
Unlike textContent, altering innerText in Internet Explorer (up to version 11 inclusive) not only removes child nodes from the element, but also permanently destroys all descendant text nodes (so it is impossible to insert the nodes again into any other element or into the same element anymore).
