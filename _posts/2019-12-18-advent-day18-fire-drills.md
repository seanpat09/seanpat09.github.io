---
layout: post
title: So-Called Advent Calendar Day 18 - Fire Drills - You have to slow down to speed up
date: '2019-12-18-T00:00:00.000-07:00'
author: Sean Cuevo
description: When you hit a P0 issue, you have to slow down first to quickly deliver

tags:
- salesforce
- agile
- bug fixes
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

<figure>
  <img src="{{site.url}}/assets/img/hydrant.jpg" alt="fire hydrant"/>
  <figcaption>Rushing into a problem generally doesn't help</figcaption>
</figure>

We have all been there before. You get the message on Slack, a panicked chat message. The dreaded P0 issue. Something is broken and we have to fix it NOW.

Generally my first instinct is to dive into the code and figure out what's wrong. Churn out the code, get it tested, merge it into master, and get it pushed to production as soon as possible. Deployments in Salesforce can take up to an hour in some of the orgs I have worked in, so we need to move quickly!

Looking back now, this is almost always the wrong way to react. Before you even touch the code, every one needs to stop and align on the problem. Set up a meeting, either in person or remotely, with everyone that needs to be involved. You may end up inviting individuals that don't really need to be involved, but given the critical nature of the issue, it's better to be overcommunicative than under.

Now take the time align everyone on the issue:

* What is not working properly? How should it be working?
* How many people are affected? What is the consequence of not fixing the problem?
* How should we should find the cause of the issue?
* Who should be working on it?
* How will we test if we have actually resolved the issue?
* Have we accounted for all use cases?
* What are the potential consequences of pushing out this fix?
* Do we need to do any communication to customers? Both during and after the fix is made?

After it is clear everyone's role needed to execute, only then should you execute. You have to slow down first and align or you risk making things even worse. And after everything has settled down, you have to do a retro on the issue with everyone involved. Here are a few questions to answer, but it should not be limited to this

* What caused the issue?
* Could we have caught this earlier? What improvements do we have to make?
* Are there any unintended consequences?
* Was there any fallout from this issue?
* Are there any actions items moving forward?

And most importantly, take a moment to praise each other for jumping on board. Regardless of how it goes, recognize the hard work involved in resolving these issues. You will undoubtedly make mistakes, but they were the best calls you could make at the time.

As a side lesson - because of the time and resources required to address the issue, before you sound that alarm, ask yourself is this really an emergency?
