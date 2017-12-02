---
layout: post
title: Configuration Driven Development
date: '2017-12-04-T07:51:00.0
author: Sean Cuevo
tags:
- salesforce
- reporting api
- configuration
- apex development
- lightning
---

When introducing people to Salesforce development, early on I usually hear something along the lines of: 

> *I don't understand what's so special about Salesforce. It just seems like an expensive database with some minor drag and drop logic*

I struggled with this idea for a long time. The slogan *Clicks not code* always irked me. At best it downplays the value of an engineer. At worst it was the cause of common anti-patterns as people would try to piece together a convoluted mixture of formulas and workflows when writing the same in Apex would be clearer and have the added benefit of being tested. In fact, when Process Builder was released, I took it as slight towards developers; they were actively working towards making developers obsolete as opposed to improving the tooling. And don't get me started on *Sassy*.

![report component](/assets/img/Saasy1.jpg)

*No software, EXCEPT FOR EVERYTHING*

I treated the configurable portions of Salesforce as a separate app that also happened to touch the same database. The way I wrote Apex would be as if I wrote Ruby without leveraging the benefits of Rails. Sure, I would use configurable components like custom metadata, field sets, and custom labels to make things a little more dynamic. And I never griped about not having to worry as much about security breaches and server maintenance. But I always felt like I was developing on top of Salesforce instead of extending it.

This changed about a year ago when I was trying to embed a report into a Lightning application. Out of the box, you can embed a report chart, but not the report itself and my users basically wanted a summary list view. My first instinct was to just build a component that displayed the results of a relatinoship SOQL query. I even thought I should store the SOQL queries as strings so you could quickly spin up new components.

But my mind kept going back to the reports. If I could just display the results of a report, a majority of the work would be complete. I wouldn't need to worry about how to model the data, just how to display the results. The result was a Lightning component that displayed the results of a summary report driven by the JSON output of the Reporting and Dashboards API.

![report component](/assets/img/report-component.png)

This was topic of my talk at Dreamforce '17, of which you can view the slides and audio recording in [this GitHub repo](github.com/seanpat09/dreamforce17).

But after building this component, the real benefits of Salesforce to an engineer became clear to me. Aside from any technical bugs that appeared, I did not have to maintain this component. An admin could manage changes using the standard report builder and easily add new report components without any help from a developer. Instead of putting in hours maintaining slight changes, I could focus on enhancing functionality.

In the upcoming weeks I'll be starting a new series of posts surrounding the idea of Configuration Driven Development in Salesforce. I'll focus on how you can start thinking with a configurable mindset, which pieces of Salesforce configuration are ripe for this style of development, and of course a series of code samples and working prototypes that you can start using! 
