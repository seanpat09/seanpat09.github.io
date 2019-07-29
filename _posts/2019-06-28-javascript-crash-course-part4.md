---
layout: post
title: A JavaScript Crash Course from a Apex Dev's Point of View - Part 4
date: '2019-06-28-T00:00:00.000-07:00'
author: Sean Cuevo
description: Learn the debugging differences between Apex and JavaScript, and how it can change your thoughts on how to debug your code.
tags:
- salesforce
- lwc
- apex
- javascript
---

*At TrailheaDX '19, I was fortunate enough to do a talk on learning JavaScript fundamentals from an Apex developer's point of view. This series covers the topics from that talk*

We (hopefully) demystifyied the `this` keyword in the <a href="{{page.previous.url}}">last post</a>. Now we'll switch gears and focus on debugging differences between Apex and JavaScript, and how it can change your thoughts on how to debug your code.

## Reactive Debugging

In Apex, generally you are doing what I like to call "reactive" debugging. Using a `System.debug` and `System.assert` calls, you generally run your code and examine the results after the fact. The [Apex Replay Debugger](https://developer.salesforce.com/blogs/2018/06/salesforce-for-vs-code-apex-replay-debugger-and-more.html) *sort of* simulates interactive debugging, but even with that tool you are looking at something that has already happened.

I often see Apex developers do the same thing when working with JavaScript using `console.log`. For example, this is a simple function that accepts an array of contacts and returns an array of the contacts' full names:

```
function getFullNames(contacts) {
    console.log(contacts);
    return contacts.map( (x) => {
        x.FirstName + ' ' + x.LastName;
    });
}
```

What I will generally see is a developer use the `console.log` function, run the code, look at the console, and repeat. Debugging like that can be helpful, but it's only a fraction of what you can do in JavaScript. Since front-end JavaScript is running on your machine as opposed to a cloud server, you have more powerful debugging tools at your disposal.

## Interactive Debugging.

Let's take the scenario when you have to debug a front-end JavaScript issue in production, so adding `console.log` calls to your source isn't really a feasible option. With JavaScript, the Chrome browser gives you tools to interact with your code while it is running without having to modify the source code.

First you need to enable [Debug Mode for Lightning Components](https://help.salesforce.com/articleView?id=aura_debug_mode.htm&type=5). By doing so, JavaScript won't be minified, making it easier to read and interact with.

```
//unminified
function getFullNames(contacts) {
    console.log(contacts);
    return contacts.map( (x) => {
        x.FirstName + ' ' + x.LastName;
    });
}

//minified
function getFullNames(e){return console.log(e),e.map(a=>{a.FirstName,a.LastName})}
```

Then, open up the Developer Tools in Chrome and go the Source tab. When you click the line numbers, you can add a breakpoint. When the code runs, the browser will actually freeze on at the line. You can see the values of variables and you when you are done you can press the play button in the top right to continue running.

![Debugger screenshot](assets/img/debugger.png)

You can also interact with the variables and even change them in the Console tab.

![Console screenshot](assets/img/console.png)

This is just some of the basic stuff you can do, so make sure to check out the `Chrome Dev Tools` documentation for more cool features.

What's fantastic about this is that this makes it easier to interact with production code. You don't have to deploy anything to debug, you can just interact with it directly in the browser. This is really useful for those scenarios where something works in a sandbox but not in production - you can debug closer to the actual problems as they are actively happening.

That concludes this crash course in JavaScript development. I hope that provides you with a foundation to continue leveraging the best of JavaScript on the Salesforce platform and you find yourself a little more fluent when reading others' JavaScript code.