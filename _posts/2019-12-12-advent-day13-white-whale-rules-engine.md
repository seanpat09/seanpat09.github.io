---
layout: post
title: So-Called Advent Calendar Day 13 - White Whale&#58; The Apex Rules Engine Part 1
date: '2019-12-13-T00:00:00.000-07:00'
author: Sean Cuevo
description: There are some projects that you never get to finish. This time it's the rules engine

tags:
- planning
- salesforce
- estimation
- agile 
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

<figure>
  <img src="{{site.url}}/assets/img/flow-diagram.jpg" alt="Drawing out a flow chart"/>
  <figcaption>Turning a flow chart into code gets tricky when you need flexibility</figcaption>
</figure>

Throughout my career I have built many MVPs that have never gone anywhere. Some of them turned out to not be as useful as we originally anticipated, but there are a select few that I think really had something to them. Either I left the company or shifting priorities prevented us from starting on the next iteration. One such product is The Rules Engine.

One of my favorite declarative features in Salesforce are Workflow Rules. Being able to define actions that run based on criteria without having to write a single line of code can be really useful, especially when that criteria isn't set in stone. This lets you respond to changes quickly with some level of confidence without having to test the underlying workflow engine every time criteria is a changed. (There is some debate to be had about how much you should test a workflow rule itself, but that's for another time).

For one job, based on a customer's contract, certain pricing was created. Not only was this based on the contract with the customer, but also based what the customer's customer's profile (e.g. account age, reward points, etc.). At first we just handled this with if statements in the code, but as the customer base grew, so did the complexity of code. Not only were there more permutations because of the increasing number of contracts, but the contracts themselves were getting more complex. Clearly if statements were not enough.

The newest customer had a particularly complex contract where customer pricing was based on the "levels" of their customers (i.e. Bronze, Silver, Gold subscription) and were only eligible for the pricing if they met certain criteria. (I'm making some of this up but it fits the general complexity of the criteria). This seemed like a good opportunity to create some kind rule engine. We first broke down the criteria into "rules". For example:

* Bronze Subscription gets 15% off IF:
  * Account is at least 3 year old
  * They have 100 reward points this year
* Silver Subscription gets 25% off IF:
  * Account is at least 2 years old
  * They have 50 reward points this year
* Gold Subscription gets 25% off IF:
  * Account is at least 1 year old

To make things more complicated, sales reps wanted to be able to override the criteria and have the discretion to just give someone the discount. So for all these rules there's an "OR if the rule is overridden for this customer". To make things even more complicated, these rules weren't set in stone, so the combination of the rules were subject to change.

The discounts for these customers were calculated in bulk so this information had to be calculated on the fly per customer. A workflow rule wasn't enough, but the evaluation of criteria in a workflow was just what we needed. I wanted to say if `1 AND 2 AND 3`, then this is person is eligible for the discount for their level.

The first thing we did was break down the logical components. Each level was like a Discount Rule and each rule had criteria that had to be met. For example, the "Bronze Subscription" was a type of Discount Rule and it had two criteria:

* Account has to be 3 years old
* Reward Points has to be at least 100

By breaking these down into logical components, an sObject model began to emerge:

* `Discount_Rule__c` object
  * `Name` - e.g. "Bronze Subscription"
  * `Discount_Amount__c`- e.g. 15%
  * `Logical_Statement__c` - holds the logic statement i.e. `1 AND 2 AND 3`

* `Criterion__c` object
  * `Discount_Rule__c` - the parent discount rule this criterion belongs to
  * `Rule_Number__c` - i.e. the number to represent this criterion in the parent logical statement field
  * `Type__c` - e.g a picklist with the values "Account Age" and "Reward Points"
  * `Condition__c` - the value that must be met for this criterion (i.e. 3 for account years, 100 for reward points)

With this structure, we could now represent our discount rules as sObject records. For example

* `Discount_Rule__c` object
  * `Name` = "Bronze Subscription"
  * `Discount_Amount__c` = 15%
  * `Logical_Statement__c` = `(1 AND 2) OR 3`

* First `Criterion__c`
  * `Discount_Rule__c` - "Bronze Subscription"
  * `Rule_Number__c` - `1`
  * `Type__c` - "Account Age"
  * `Condition__c` - 3

* Second `Criterion__c`
  * `Discount_Rule__c` - "Bronze Subscription"
  * `Rule_Number__c` - `2`
  * `Type__c` - "Reward Points"
  * `Condition__c` - 100

* Second `Criterion__c`
  * `Discount_Rule__c` - "Bronze Subscription"
  * `Rule_Number__c` - `3`
  * `Type__c` - "Override"
  * `Condition__c` - true

Now that we have our rules, we now need to create the engine that would take a customer, evaluating them based on this criteria and their subscription level, and then return the appropriate discount.

In the next part, we'll discuss how I used the [Composite Pattern](https://developer.salesforce.com/page/Apex_Design_Patterns_-_Composite) to build the engine with Apex.