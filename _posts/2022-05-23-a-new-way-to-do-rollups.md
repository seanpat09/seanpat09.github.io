---
layout: post
title: "Can we trick salesforce into doing reportable dynamic rollups?"
date: "2022-05-23-T00:00:00.000-08:00"
author: Sean Cuevo
description: Misusing External Objects to Do Dynamic Rollup Summaries
image: /assets/img/rollup-summary-report.png

tags:
  - rollup-summary
  - apex
---

<figure>
  <img src="{{site.url}}/assets/img/rollup-summary-report.png" alt=""/>
  <figcaption>
    Make reportable rollup summaries by "misusing" the apex connector framework.
  </figcaption>
</figure>

## Rollups. Why is it always rollups.

I hate rollups. I hate building them. I hate how they slow down things in Salesforce. I hate how they can accidentally block users from editing records. And I especially hate debugging them. "Why is this roll up field wrong?" I don't know, there are so many points of failure when it comes to a roll up. Maybe the job hasn't run yet and someone just added another record. Maybe an error was thrown and a record wasn't accounted for. But sure, I'll throw away a few hours to answer your question and by the time I have an answer the rollup corrected itself. I digress.

And I say all this as a contributor to the Declarative Rollup Summary open source project. I get why we need rollups. Not every org has access to an analytics tool and is limited to Salesforce reporting. And sometimes an orgs needs to answer the question "How many Account records have donated more than $1000?" and the person asking the question needs to do this with a report. So you build a rollup summary because the alternative is building a custom reporting tool and that's it's own set of problems. That's why DLRS is such wonderful tool. I just hate it that it has to be like this. Or does it?

## SOQL to the rescue

What infuriated me for years was that SOQL can actually do this for you with aggregate queries. For example, if you captured your donations for an Account in a child object called Donations, you could do the following with SOQL:

```
SELECT Account__c, SUM(Amount__c) FROM Donation__c GROUP BY Account__c HAVING SUM(Amount__c) > 1000
```

But the goal here is to make this reportable. Sure, you might get away with having devs and admins run these SOQL queries for the business, but part of scalability is providing a mechanism for self-service. So you make the rollup field.

## Enter the Apex Connector Framework

I was playing with the [Apex Connector Framework](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_connector_start.htm) to pull some data into Salesforce from an API so that they could be reportable. The flow of this framework is as follows

* Define fields on an external object (i.e. how will this object be represented in Salesforce?)
* Extend the framework classes, which mainly allow you to interpret a SOQL query, sent the search criteria to an API with a callout
* Parse the results and map to rows for the external object.

After you do this, the external object becomes reportable! So I thought, what if instead of doing a callout we just did the aggregate query from above and mapped those results to the external object. In other words, I have an external object called `AccountRollup__x` with a field `TotalDonations__c`, and every time it is queried, it does an aggregate of Account records?

Turns out, it kind of works and it's reportable! You can run a report using filters on the `TotalDonations` field as if you were reporting on a regular rollup field.

See this repo for code samples of a prototype that does just this you can try in a scratch org: [https://github.com/seanpat09/live-rollups](https://github.com/seanpat09/live-rollups)

## Caveats
I ran into a few issues while building this prototype.

* For some reason, I could not display fields related Account fields on the report as I was getting a system error with a gack. That's why I suggest linking to the summary field in the report with the Name column, which can give you access to the Account via the `AccountRecord` lookup field
* When I tried to make the rollup summary record a parent to the Account, the report would just break, not allowing me to do the filtering at all
* You're limited to whatever SOQL limits you are normally limited to, so if you have a really high volume org, this might not work at all.
* I'm not sure of the licensing on the Apex Connector Framework, but I was able to use it in an Enterprise org that does NOT have the license for Salesforce Connect.

So what does this mean? I'm not really sure yet to be honest. But if this works and scales scales, it might be another option to rollup summaries without having to run background jobs to maintain the data! Instead of debugging a job, you're debugging SOQL queries instead, which might be much easier to maintain for your team.

