---
layout: post
title: "Technical Debt Is The Wrong Term - We Should Call it Code Rot"
date: "2022-12-12-T00:00:00.000-08:00"
author: Sean Cuevo
description: Why we should stop using the term technical debt

tags:
  - technical debt
---

## What is technical debt?

The term technical debt gets thrown around at any company that maintains any sort of code base, often used to explain to stakeholders the need to dedicate resources to refactoring. Generally I hear it summarized as expediting delivery by intentionally writing imperfect code, thus saving time. Eventually this debt has to be paid back eventually - either in development time to address it proactively or in the time to recover when things go wrong. However, financial debt has always felt like a poor comparison and at best miscommunicates the problem and at worst misleads the audience.

## Financial debt is a bad analogy
Financial debt has little in common with what we call technical debt. Financial debt has to be paid back eventually, usually in a very defined schedule with specific consequences for not doing so. Code is not like that all; there is no "due date" for technical debt and it may never come. The internet is held together with scripts written in a weekend that have been running for years. You also usually don't know the consequences of not refactoring. Sometimes the market for the product just ends and the code gets thrown away, never needing to be refactored.

There is also an element of intentionality when it comes to financial debt. You take out a loan with the goal of accomplishing something that has a higher rate of return than the interest on the loan (well, in theory, but that's another blog post). With code, there are occasions where you intentionally make that decision. For example, I often work on an MVP and see an opportunity to use a design pattern to make a feature more scalable, but opt not to go that path because solving for scale isn't needed at the time. If that need for scale comes suddenly, it will be harder to implement on that tighter timeline in the future, but we don't handle it now because if we don't deliver, scale won't be a problem.  This is what I think most of people consider to be technical debt. However, most technical debt is not intentional and as such we shouldn't call it debt. We should call it "rot."

## Code Rot
The second code is delivered it has an expiration which could come due to known future scenarios and unknown external influences. For example:

* A security vulnerability could be discovered, requiring a rewrite several pieces of functionality
* Version upgrades could deprecate functionality that you depended on, requiring a work around
* Your product could start to be used in unexpected ways, creating bottlenecks in unexpected places
* Even lack of documentation causes code rot, expiring immediately the day the one person that understands everything decides to leave. (There's no such thing as legacy code, just code that you didn't write and now have to maintain!)
* You could have just plain not anticipated something. It happens all the time and should be expected

And kind of like food, there's no defined expiration date. Sure, you know that the milk will go bad after a few weeks, but how is the flour after a year? And honey doesn't go bad, right? If you aren't careful, one day you will open the fridge and it will reek and you'll have no idea where it's coming from.

## Keep your code fresh
If you want to keep your code fresh, you need to look through the fridge regularly. A really good practice is to get rid of functionality you don't need. Code that doesn't exist doesn't go bad! The idea of "what's the harm in keeping it?" is a poor one because all code must be maintained.

Log everything. You should have technical notes for each of your features, either in comments or in separate documentation. Call out places that are potential technical debt so that future people (including yourself) have an idea why something is the way it is.

Speaking of technical documentation, make sure it exists! I am a huge fan of code comments on methods explaining what's going on and the intent. Runbooks are great too, i.e. documentation of how a feature is expected to be used. Keep your solution docs archived and make sure you have a system where your commits can be linked to a change request. Historical context is key to making sure you are addressing rot correctly.

## Code rot is everyone's problem
I often see technical debt used as a way to deflect on a problem - with technical stakeholders saying they don't get enough time and resources to address issues and non-technical stakeholders saying they don't have enough metrics to measure success. Cleaning out the code fridge is everyone's problem.

If you are technical contributer, it is on you to track where the rot is, what needs to be done, and take a swing at some predications on the consequences of not doing so. If you're a non-technical contributer, you need to consider that this time is table stakes for successfully delivering a product and be ruthless when it comes to prioritizing new work. If you cannot tell the technical stakeholders what the consequences of not adding a feature is, then the feature doesn't need to be added.

This is all easier said than done - this is both a technical problem and a cultural one. I'm hoping if that if we start using a term that better fits the reality, then maybe we'll be more successful at communicating it's importance to the code we work on.