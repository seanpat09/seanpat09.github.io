---
layout: post
title: So-Called Advent Calendar Day 10 - Should vs. Does
date: '2019-12-10-T00:00:00.000-07:00'
author: Sean Cuevo
description: Just because code functions a certain way does not mean it should

tags:
- code
- salesforce
- comments
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

I have worked in numerous Salesforce orgs and whenever I walk into a new org, I try to make an effort to audit the code to get a feel for the product. I figure, the application can only do what the code tells it to do, so the code is ultimate source of truth of the application's functionality, right?

Technically that's true, but just because an application functions a certain way doesn't mean it *should* function that way. If it did, then there would never be any bugs! Let's take an example of a piece of code that calculates some tax:

```
public Double calculateTax(Double income) {
    return income * 0.25;
}
```

With just that alone, you would think "OK, the tax rate is 25% and we're using a flat tax." While that is technically true, that might not be the intention of how product wants the tax to be calculated. Now let's look a the same method but with a comment:

```
/*
 * @description Calculates tax using current US tax brackets
 * @returns double
 */
public Double calculateTax(Double income) {
    return income * 0.25;
}
```

Clearly that description is out of sync with what the method is actually doing and without it we would be none the wiser. It doesn't necessarily have to be a comment, but the intended behavior needs to documented somwhere.

I used to believe that there should NEVER be comments because the code should document itself. But every product I have worked on isn't just made by developers writing code. It's a collaboration between product managers, stakeholders, customer feedback, and developers working together.

I actually don't know what the ideal best practice is for documentation. Is it comments in the code? Extensive documentation managed outside of the code base? Through the project tracker? User guides? That's up to your team, but what I know for sure is that code that is left to document itself is always incomplete. Documentation is the metadata for the functionality of the application and it needs to be easily accessible by everyone collaborating, not just hidden inside the code base.

So the next time you inherit a legacy code base, ask for any documentation on the intended behavior exists. If that documentation does not exist, your code audit shouldn't define your application's behavior - it should just serve as a starting place on clarifying what your application should do, as opposed to what it actually does.