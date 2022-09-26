---
layout: post
title: So-Called Advent Calendar Day 4 - Virtual Mocking
date: '2019-12-04-T00:00:00.000-07:00'
author: Sean Cuevo
description: Virtual methods can be a great tool when you need to stub out a small part of test
tags:
- programming
- Salesforce
- interface
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

When writing unit tests, I try to avoid doing anything that would hit the database, like creating records, or running SOQL queries. Inserting records in tests not only slow tests down, but they also inadvertently create a dependency between your test class and any triggers in your organization. Or in the case of BigObjects, you just can't insert recors because they'll persist in your org even after the test is done (we'll discuss this on another day!). While you will want a larger integration test that actually hits the database to test things from end to end, generally your unit tests are more focused and are also more in number. So even if just for the sake of speed, I try to think if I can write unit tests without any DML at all.

This becomes especially tricky if the method you are testing has any SOQL queries in it. While using dependency injection or the StubAPI can be useful to mock out services that do DML, sometimes it's a bit too much to create a separate dependency for your method. Consider the following that queries some objects and returns the first one to match some criteria: 

```
public with sharing class Finder {
    public SObject findTheThing() {
        List<SObject> things = [SELECT Name FROM Thing__c];

        for(SObject thing : things) {
            if(isThisTheThing(thing)) {
                return thing;
            }
        }
        return null;
    }

    private Boolean isThisTheThing(SObject theThing) {
        return theThing.get('Name') == 'The thing';
    }
```

Because of that SOQL query in `findTheThing`, it seems like you have to insert a record to test the positive use case where a match is found. I want to avoid hitting the database, however it doesn't seem right to add another class in here to mock out the SOQL query. That's where virtual classes can be handy. Let's rewrite the class above to move the SOQL query into it's own separate virtual method:

```
public with sharing virtual class Finder {
    public SObject findTheThing() {
        List<SObject> things = getTheThings();

        for(SObject thing : things) {
            if(isThisTheThing(thing)) {
                return thing;
            }
        }
    }

    private Boolean isThisTheThing(SObject theThing) {
        return theThing.get('Name') == 'The thing';
    }

    protected virtual List<SObject> getTheThings() {
        return [SELECT Name FROM Thing__c];
    }
}
```

With a virtual class, I can create an extension of the class in my test that mocks out the virtual method.


```
@isTest
private with sharing class Finder_TEST {

    @isTest
    private static void shouldFindTheThing() {
        SObject expectedThing = new FinderMock().findTheThing();
        System.assertEquals('The thing', expectedThing.get('Name'), 'The correct thing should be returned');
    }

    private class FinderMock extends Finder {
        protected override List<SObject> getTheThings() {
            return new List<SObject> {
                new Thing__c(Name = 'The thing'),
                new Thing__c(Name = 'Not The Thing'),
            }
        }
    }
}
```

By extending the virtual `Finder` class I am able to override the virtual `getTheThings` method while the rest of the logic is kept in place. Now I can test the `findTheThing` method without having to insert any records and the test runs super fast! I could also do the same for the negative case by just altering the list of records that gets returned by the mock.

For these unit tests, I don't care about the actual SOQL query because the tests are focused on the matching logic, not that the SOQL query ran correctly. I should certainly add a test that does hit the database to test the SOQL query, but by using mocking the virtual method I only have to hit the database for the tests that require it instead of incidentally for every test. By adopting this pattern, your test suite will run much faster and that's a little less time you have to spend just staring at that deployment circle, waiting for that green checkmark.