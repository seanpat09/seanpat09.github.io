---
layout: post
title: So-Called Advent Calendar Day 3 - Retro Forgiveness
date: '2019-12-03-T00:00:00.000-07:00'
author: Sean Cuevo
description: When it comes to evaluating legacy code you should default to forgiveness, even when it was written by you
tags:
- programming
- Salesforce
- interface
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

<figure>
  <img src="{{site.url}}/assets/img/difficult-roads3.jpg" alt="Difficult roads lead to beautiful destinations"/>
  <figcaption>Where we can go depends on where we have been, good and bad</figcaption>
</figure>

Retro is one of my favorite scrum ceremonies and I think it is the most important one. Without taking the time to reflect and celebrate what we have done, we cannot possibly set ourselves up for a better future. On my current team, we start with the Retro Prime Directive:

> ~~The Prime Directive prohibits Starfleet personnel and spacecraft from interfering in the normal development of any society, and mandates that any Starfleet vessel or crew member is expendable to prevent violation of this rule~~

I'm sorry, that's the wrong one.

> "Regardless of what we discover, we understand and truly believe that everyone did the best job they could, given what they knew at the time, their skills and abilities, the resources available, and the situation at hand."
--Norm Kerth, Project Retrospectives: A Handbook for Team Review

I love how this quote encourages you to frame your perspectively positively before reflecting on the past. How many times have you looked at some legacy code or some existing solution and thought "What the fuck was this person thinking?" That only an incompetent developer would build something like this, build something that you now have to fix? And how many times has that rage actually helped you solve the problem?

The Retro Prime Directive encourages you to avoid all of that and to instead consider the context of the situation.

Maybe there was external pressure that forced them to rush through a solution.

Maybe the "right" way to do it causes some hidden side effects and the "shit code" was the best solution they could come up with at the time to get around it.

And maybe they just straight up didn't know any other way, but they were the only person there to could do it.

We have all been that past author that wrote the legacy code someone else had to fix. You deserve to be forgiven and you should afford others that forgiveness as well.

This is easier said than done. It is so much easier to complain about legacy code and admittedly a little more fun. But when you focus on blame, you get stuck in the past. Instead, I encourage you to focus on seeing what you can learn to help guide what you build moving forward.