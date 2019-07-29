---
layout: post
title: Stop Fixing My Features!
date: '2019-07-29-T00:00:00.000-07:00'
author: Sean Cuevo
description: Sometimes fixing a bug can have unintended side effects when a customer is relying on it as a feature
tags:
- salesforce
- refactoring
- apex
- bugs
---

When I hear the phrase "That's not a bug, that's a feature!" I typically imagine that the speaker is a developer defending their code. But sometimes it's the end user that's taking advantage of that bug to get the system to do what they want.


## Intention vs. Actual Use

Consider the following pseudocode for validating an amount on a line item:

```
if (lineItem.value == null || lineIem.value < 0) {
    throw LineItemException('Line item value must be populated and cannot be 0 or less');
}
```

The if statement only prevents values that are `null` or less than 0, but it actually allows a value of 0. The exception message, however, indicates that 0 is also invalid. To me, that exception message indicates the true intention of the code, meaning there must be a bug in the if statement. Noticing this, I went ahead and fixed the if statement to include 0:

```
if (lineItem.value == null || lineIem.value <= 0)
```

However, when this code was released, a bug was reported from the end user.

*Something changed in the last release, I can no longer add line items with a 0 value. We've been doing this to adjust Opportunities that went from Closed Won to Closed Lost so that we can capture the fact that the line items were part of a deal that happened, but didn't actually get sold.*

My first instinct was to respond with "No, you can't use it like that, you need to change your process." This is also probably why I am not a product manager. But that's not necessarily the correct response. That customer may have thousands of line items like this and this process may be ingrained in their training. They may have built customization that relies on this "feature".

Just because a bug exists doesn't necessarily means you should fix it. Imagine if Salesforce decided to completely disable [URL hacking](https://www.salesforceben.com/salesforce-url-hacking-tutorial/). Sure, they may not be officially supported, but they are used by countless orgs! I've seen at least one in every single Salesforce org I have ever worked in. So while it's not necessarily a "feature", it has become one that customers rely on. So when you should fix a bug?

## To Fix or Not To Fix?
Before fixing a bug, you should do some research as to how pervasive the bug is. Here are some questions I like to ask myself when it comes to fixing bugs:

* What will break as a result of this bug? i.e. What would break if line items had a value of 0?
* How pervasive is the bug? i.e. How many line items actually have a value of 0?
* Is there a work around for this bug or does it create a blocker?
* How many people does this bug effect?

And now, something I didn't ask before:

* What would break if we DO fix this bug?

It's an uncomfortable question as the answer can go against the vision of what your product should do. Do you fix the bug and risk angering the customer or do you leave the bug and possibly create future constraints on what the product can do in the future? Of course, the answer will vary from case to case as you weigh out each alternative. But next time you find yourself thinking you should quickly fix something that you run across, stop and ask yourself if someone thinks that bug is a feature.

