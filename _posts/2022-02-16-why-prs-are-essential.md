---
layout: post
title: "Pull Requests are an Essential Part of Security"
date: '2022-02-16-T00:00:00.000-08:00'
author: Sean Cuevo
description: Pull requests aren't just a great part of collaboration, but also serve as an essential part of security
image: /assets/img/john-schnobrich-FlPc9_VocJ4-unsplash.jpg

tags:
- work life balance
---

## Separation of Duties

Before I became a software engineer, I studied accounting and even worked as an auditor for a brief period. Not much accounting knowledge remains, but a few nuggets stayed with me, particularly the internal control known as "Separation of Duties" (not to be confused with the design principle "Separation of Concerns"). To summarize, no one person should have complete control over an entire transaction. For example, the person collecting cash from a customer for a bill should not be the one to also record the transaction and deposit the cash. Otherwise, that person could collect funds, credit the customer's account, and take a little off the top before making the bank deposit. Having another person in this process does not eliminate the risk, but colluding with someone else is harder to hide. You know what they say about the only way for two people keep a secret.

What does this have to with code and pull requests? Pull requests serve a similar security purpose. Not only do pull requests provide the nice to haves of allowing for asynchronous collaboration, a second set of eyes, etc, they are also a very important auditing artifact and should be considered an essential component of your delivery process.

Code controls business logic, often at a pace magnitudes faster than what any human can keep up with. And because of the complex nature of writing code, it poses a huge security vulnerability where malicious behavior can easily be obfuscated. For example a [fork bomb](https://en.wikipedia.org/wiki/Fork_bomb) looks like an inconspicuous line of code, but can actually crash your entire system. Because of which, every line of code that makes its way into any system of importance should be reviewed by someone else. Pull requests not only provide the mechanism to do this review, but also create the audit trail for every change.

## Components of a Pull Request
Let's examine the components of a pull requests and how they contribute to security.

First and foremost a pull request aggregates proposed changes via its commits. By using `git blame` you are able to find the latest commit attributed to every line in the codebase. With this you are able to follow the history of any line change. If you ever wondered why a change was made, following the commit history is a great start. With GitHub, you can even see which pull request was the one that brought that commit into the current branch.

Not only can you see the history of changes, but the commits attribute the changes to the person that introduced it. Commit history great, but also being able to talk to the person that made the change can help provide more context that's not in the code.

So far these are just things you get from a commit history alone. Pull requests show their value in the additional layer of tracking they provide. With a pull request, we can see the reviewer that approved the merge. Now you have at least two people involved in the process of getting code into production (provided that you have configured your version control to not allow self-approvals).

The pull request itself also provides an area to have a discussion - on the features, the code decisions, tech debt introduction. By doing this in the pull request, this discussion is linked very closely to the code itself. You could have the discussion happen in some other document, but several months down the line that document may get lost to some obscure folder in your company's google drive.

For extra credit, you could also tag pull requests with the work item associated to the pull request, so you know why a change was made. 

## Security Benefits and Beyond
With all of these ingredients together, you can trace from a single line change who made the change, approved the change, approved the item to be worked on at all, and any context capture in any comments for the PR! And with all theses duties separated by person, you've mitigated a lot of the risk of any one person trying to act maliciously within an organization.

As an auditor, this kind of paper trail is gold. In fact, I've been involved in security audits that went super smoothly because this process was in place. In addition to security benefits, it was so useful to be able to see the context of something far into the future. Have you ever looked at code that YOU wrote and wondered why you took some odd approach? With this trail you can find the context of the original change, hopefully with an explanation of why.


## Constructive Review vs. Fodder for Punishment
I want to emphasize that this internal control is not about placing blame. Accountability is important, but if this process is wielded as a threat, your team will be reluctant to to adhere to it. Should something happen, a bug, security breach, or even just a missed feature, this process should be used to trace the source of the problem. Afterwards a constructive, blameless discussion can happen where processes can improved. What happened in the past happened - the only thing to do is move on and learn from it. If leadership uses this process to punish contributors for their mistakes, the engineers will find ways to not be associated to the changes. Like any tool, its value to the organization highly depends on the culture those using it.

There are not many things that I consider essential for an engineering org, but pull requests are something that is a must have for me, and possibly a deal breaker for me if an organization refused to use them without a much better alternative. Even when I was the only working on a project professionally, I STILL used pull requests so that I could have the benefits of the audit trail, both for my future self or anyone else who would have to inherit my work after I left.



