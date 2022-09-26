---
layout: post
title: So-Called Advent Calendar Day 20 - Securing Your SOQL
date: '2019-12-20-T00:00:00.000-07:00'
author: Sean Cuevo
description: New ways to secure your SOQL

tags:
- soql
- apex
- security
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

A common misconception I have heard is that using `with sharing` when declaring an Apex class enforces security in your code. However, that only enforces sharing rules, i.e. it prevents the running user from querying and updating records that they do not have sharing access.

It DOES NOT enforce FLS. Even if the running user does not have read/edit access to a field on an sObject, when that field is queried in Apex, that field will be populated. Using `<apex:outputField>` in Visualforce will prevent the user from seeing the value, but the value is still fetched in apex.

There are two security mechanisms in Apex that are in beta that can really help you enforce field level security in your code.

[The SOQL clause WITH SECURITY_ENFORCED](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_with_security_enforced.htm)<br/>
[The stripInaccessible method](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_classes_with_security_stripInaccessible.htm)

Salesforce documentation does a great job outline how they work, but they are not interchangeable.

## WITH SECURITY_ENFORCED
When the clause `WITH SECURITY_ENFORCED` is added to SOQL query, if the running user does not Read access to any of fields in the `SELECT` statement, an exception will be thrown. For example

```
List<Account> accounts = [SELECT Website FROM Account WITH SECURITY_ENFORCED];
```

This query will throw an error if the running user does not have Read access to the `Account.Website` field. It only applies to the `SELECT` statement. For example:

```
List<Account> accounts = [SELECT Id FROM Account WHERE Website = 'socalledprogrammer.com' WITH SECURITY_ENFORCED];
```

Since `Account.Website` is not included in the `SELECT` statement, this won't throw an error even if the running user doesn't haven't Read access to the field.

So with a single statement, you can start enforcing field level security in your queries. However, the downside is that the query fails hard and the error message does tell you what field is needed. Also, I have seen it used with queries that try to emulate `SELECT *`, which means that EVERY field on the object needs to be readable by the running user in order to run at all. So while the query is secure, to get the app to run at all you'd have to grant access to every field on the object, defeating your security effort. Luckily Salesforce provides an alternative.

## The .stripInaccessible method
The `stripInaccessible` method is a little more nuanced and does the check after the query is made. Here's the example from Salesforce's documentation:

```
List<Account> accounts = [SELECT Name, Website FROM Account];
SObjectAccessDecision decision = Security.stripInaccessible(AccessType.READABLE, accounts);

for (Integer i = 0; i < accountsWithContacts.size(); i++) 
{
    System.debug('Insecure record access: '+accountsWithContacts[i]);
    System.debug('Secure record access: '+decision.getRecords()[i]);
}

// Print modified indexes
  System.debug('Records modified by stripInaccessible: '+decision.getModifiedIndexes());

// Print removed fields
  System.debug('Fields removed by stripInaccessible: '+decision.getRemovedFields());
```

This gives you a little more control because you make a decision based on the results of the security check. For example, you could log an error for an admin to check to later to see if there are missing permissions, or if someone is trying to see something they aren't supposed to.

To be honest, I can't think of a really good reason to use WITH SECURITY_ENFORCED. The error it throws is too vague and it makes it really hard to debug. But it's good to know there is a quick option to bail hard on a query if the running user doesn't have the necessary permissions.

These items are still in beta, but I would give them a try. Security often gets forgotten when writing Apex, so it's nice to have additional easy to use tools to make things a little more secure in your code.

