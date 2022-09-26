---
layout: post
title: So-Called Advent Calendar Day 11 - Testing as Documentation
date: '2019-12-11-T00:00:00.000-07:00'
author: Sean Cuevo
description: Unit tests are a great way to link your code to expected behavior

tags:
- code
- salesforce
- comments
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

Yesterday we talked about documenting your application's functionality outside of your code. But someone pointed out to me that your automated tests are also a form of documentation. Your automated test suite, whether it is Apex tests, LWC Jest Tests, or something like Seleniun, should not just test that the code runs but also validate the expected behavior. And a robust test suite should clearly describe those test scenarios.

I really like the format Jest uses when testing LWC. The describe blocks give you the freedom to describe each scenario clearly and the expected behavior within each scenario:

```
describe('Registering as a new user', () => {
    it('should have the correct label and name', () => {
        expect(lwcInput.label).toBe('Name');
        expect(lwcInput.name).toBe('Name');
    });
});
```

And when the tests run, you get a clear message as to what exactly is malfunctioning.

In Apex, you don't quite have as many options. This is totally up to your team, but I try to mimic these describe blocks by using the method names of the tests:

```
@isTest
private static void shouldRegisterANewUser() {
    Boolean result = MyPage.register('New Username');
    System.assertEquals(true, result, 'The new user should be registered' );
}

@isTest
private static void shouldNotifyUserIfUsernameIsTaken() {
    Boolean result = MyPage.register('Existing Username');
    System.assertEquals(false, result, 'The user should not be registered' );
}
```

It's a little crude, but it does make the test results more readable. By looking at the failed method name, you get a glimpse of the scenario that's not working. It's definitely a lot better than using generic method names and no assertions, an anti-pattern I have seen in way too many orgs:

```
@isTest
private static void test1() {
    Boolean result = MyPage.register('Existing Username');
}

@isTest
private static void test2() {
    Boolean result = MyPage.register('New Username');
}
```

With generic method names, you can't really tell what's being tested unless you go walkthrough the entire test method. The lack of assertions means you're not really ever sure what the expected behavior should be. And without that information, just because the test checkmark is green doesn't mean you have met the test's intended criteria.