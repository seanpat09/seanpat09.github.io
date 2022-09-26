---
layout: post
title: So-Called Advent Calendar Day 5 - With Sharing is Caring
date: '2019-12-05-T00:00:00.000-07:00'
author: Sean Cuevo
description: An embarassing lesson in Salesforce security.
tags:
- programming
- Salesforce
- interface
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

For a long time, I was under the impression that creating an Apex class with the `with sharing` declaration was enough to enforce not just sharing rules, but also field level security (i.e. FLS) and CRUD access on objects. This was (mistakenly) reinforced by the fact that if I was trying to edit a record in a Visualforce page and couldn't, then running the class `without sharing` usually "fixed" that. But that wasn't because my user didn't have the correct CRUD access on said record, it was because they didn't haven't the correct sharing access to edit *that particular record*.

I've worked in a lot of Salesforce orgs where record access wasn't kept really private - basically everyone was a step down from being a system admin. All records were accessible, everyone had the ability to read or edit any field. And so for an embarassingly long time I didn't actually need to worry about how security worked. As long as record access was kept behind a login into their Salesforce org, the client often thought that was secure enough (for better or worse).

So here's my current understanding of record security in Salesforce, hopefully this will be insightful for you as well. *AND PLEASE CORRECT ME IF I'M STILL WRONG BECAUSE I REALLY NEED TO KNOW.*

**Sharing** - This controls whether or not a User can Read or Edit a particular SObject record (i.e. a particular Account record of all the. Org wide defaults create a baseline of access that can be expanded via sharing rules, manual sharing, or hierarchies. One thing to note is that "Write" sharing access refers to the ability to update a particular record. Create access is not controlled by sharing rules. 
`with sharing` enforces these rules in Apex. `without sharing` allows Apex to bypass sharing access and access whatever values that are in the system regardless of the running user.

**CRUD** - This stands for Create, Read, Update, Delete. This defines whether a user has the ability to do those for actions for a particular SObject at all. If you don't have Update access on Account, even if Account has a org wide default of `Public Read/Write`, you still won't be able to update any accounts. This also works the other way around. If you have update access on Account, but Account has an org wide default of `Private` or `Public Read`, you only have the ability to update an Account you have the appropriate write sharing access to. Because Apex runs in a system context, this is not enforced unless you manually enforce it by checking permissions before doing DML! `with sharing` does NOT enforce this!

**FLS** - This stands for Field Level Security and controls a User's ability to Read/Edit a particular field. This is superceded by CRUD access, i.e. if you do not have Edit access to an SObject, you won't be able to edit a field even if you have edit access on that particular field. Similar to CRUD, this is also not enforced in Apex! Apex can still read fields returned by a SOQL query even if the running user does not have read access to those fields. (Ways to enforce this will be discussed on another Advent day!)

I hope that's both correct and straightfoward enough to clarify permissions to anyone who is still confused.