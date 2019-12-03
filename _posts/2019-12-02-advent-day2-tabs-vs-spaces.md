---
layout: post
title: So-Called Advent Calendar Day 2 - Tabs vs. Spaces
date: '2019-12-01-T00:00:00.000-07:00'
author: Sean Cuevo
description: I can't settle the debate here, but it does actually matter
tags:
- programming
- Salesforce
- interface
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

<figure>
  <img src="{{site.url}}/assets/img/whitespace.png" alt="Code sample with mixed whitespace"/>
  <figcaption>Whitespace matters because readability matters</figcaption>
</figure>

On every engineering team I have ever been on, eventually the "Tabs vs. Spaces" debate comes up with strong opinions from both sides. For those who are unaware, the basic argument boils down to how should a `tab` be represented in code? As 4 space characters, or a single tab character? So today we explore....

## *Tab vs. Spaces: What does your choice say about you? You won't believe number 4!*

Just kidding - to me, it actually doesn't matter. But what does matter is the underlying issue addressed in the discussion: code readability. Whether you are on team spaces or team tabs (go spaces!), both sides agree that you have to choose a side - mixing the two is not an option. In most text editors, a tab character and 4 space characters take up the same amount of physical space on your screen. However, some tools do not follow that trend - for example GitHub renders a tab using 8 spaces.

If you mix your tabs and spaces, it may look fine on your editor, but when you go to review it on GitHub, your formatting goes out the window. And that's why the argument matters - code needs to be readable by anyone who works on it, not just those who have the same setup as your IDE.

So what should you do?

* Make your whitespace visible on your IDE. That way you can quickly identify when you have mixed whitespace
* Choose a side! It doesn't matter which, but just choose one that your team agrees upon and then never discuss it again. If you can, make it a part of your commit process to automatically format your code so no one has to think about it.
* If you can't choose a project wide rule, then at least be consistent within each file. If a file is using tabs, keep using tabs. If it's using spaces, keep using spaces.
* If you want to fix a file, make a commit that is just a whitespace commit before you make your changes. That way, when someone reviews the pull request it's easier to see what changes you actually made vs. which lines were touched only because of the whitespace changes.

So remember, it doesn't matter what side you are on as long as you are consistent! And consistent code is readable code and that's what matters the most!

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>



...is everyone gone?

The correct answer is spaces. A space is a space everywhere and you never have to worry about it formatting differently. Also if you use tabs, you will still need to use spaces and will end up mixing the two anyway. This is especially true for any line of code that you need to break up into multiple lines and with tabs you won't be able to align things nicely. And if you are worried about the number of characters, then you should be minifying your code and then this won't matter anyway.

See you tomorrow!