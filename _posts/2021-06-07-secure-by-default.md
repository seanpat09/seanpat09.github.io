---
layout: post
title: Secure By Default
date: '2021-06-07-T00:00:00.000-07:00'
author: Sean Cuevo
description: Enforcing security without nuance can actually create insecurity

tags:
- security
- apex
---

Far too often I see code where security is treated more like a checkbox instead of as part of the architecture of the product. As long as my Apex classes are running `with sharing` and I enforce FLS/CRUD access in my SOQL queries and DML statements, my code is secure, right? Technically that's true, but as is always the case, it really depends on the context.

## Tell me if you heard this one before

Imagine a manufacturing company called "Can It Already!" that produces the cans for canned food (can you?) and they use Salesforce to manage their customer orders. The account team wants to make sure that customers are called by the same account executives in order to build a relationship. To accomodate tracking this, they have requested a few fields on the Account object.

* The last executive to successfully call the customer.
* When that last call took place.
* The outcome of that call.

For the sake of this post, let's pretend that adding these fields is the best solution (it's not). The account team logs all of their calls as Tasks, so you figure a Task trigger is the way to go here. Below is the service class you build that gets called by the trigger:


```
public with sharing class AccountCallLoggingService {
  public void logCallOnAccount(Task call) {
    if (call.Status != 'Success' || !hasAccessToAccount(call)) {
      return;
    }

    accountToUpdate.LastCaller__c = call.OwnerId;
    accountToUpdate.LastCallStatus__c = call.Status;
    accountToUpdate.LastCalledOn__c = DateTime.now();

    update accountToUpdate;
  }

  private Boolean hasAccessToAccount(Task call){
    UserRecordAccess accountAccess = [
      SELECT RecordId, HasEditAccess
      FROM UserRecordAccess
      WHERE
        UserId = :UserInfo.getUserId()
        AND RecordId = :call.WhatId
    ];

    return
      Schema.sObjectType.Account.fields.LastCaller__c.isUpdateable()
      && Schema.sObjectType.Account.fields.LastCallStatus__c.isUpdateable()
      && Schema.sObjectType.Account.fields.LastCalledOn__c.isUpdateable()
      && accountAccess.HasEditAccess;
  }
}
```

## A 10 foot wall with a wide open gate

This service will only update an Account if the running user has update access on the appropriate fields and on the Account itself. This looks pretty secure, right? The code is enforcing security checks, but in practice this might have some unintended consequences.

By enforcing FLS/CRUD and sharing access on the Account record, the account executive (i.e. the running user) must now have that corresponding access. Granting that access, however, gives the account executive those permissions not only in the context of this trigger, but also everywhere in the CRM. The business may not want an account executive to be able to go to an Account record and update those fields manually, or to even be able to update the Account record at all. So while the code checks security access, have you really made the application more secure?

To be clear, this is not necessarily wrong - the problem comes with blindly enforcing security without regard to your user's experience and inadvertantly creating a security vulnerability. Enforcing frequent password changes may sound like a better security model, but [actually makes security worse](https://arstechnica.com/information-technology/2016/08/frequent-password-changes-are-the-enemy-of-security-ftc-technologist-says/). Similarly, enforcing security checks everywhere in your code may sound more secure, but in practice your users may have to be granted more permissions just so your system is usuable.

The worst example of this I have seen this is in managed packages that are trying to pass security review and throw `with sharing` into every class and `WITH SECURITY_ENFORCED` into every query. The code might pass security review, but without a nuanced approach, the customer is forced to choose between not being able to use the functionality, or having to expand their FLS/CRUD and sharing settings to a model that accommodates the code base instead of the organization.

## Running user vs System user

With this in mind, let's revist our service code with some adjustments:

```
public without sharing class AccountCallLoggingService {
  public void logCallOnAccount(Task call) {
    if (call.Status != 'Success') {
      return;
    }

    accountToUpdate.LastCaller__c = call.OwnerId;
    accountToUpdate.LastCallStatus__c = call.Status;
    accountToUpdate.LastCalledOn__c = DateTime.now();

    update accountToUpdate;
  }
}
```

This code is running `without sharing` and doesn't check the running user's access. Generally this would set off some alarm bells, but I would argue that this is more secure. This functionality is more of a system level operation that should run the same regardless of the running user. I would rather escalate the running user's permissions in this isolated context instead of being forced to grant that access to the user across the entire system.

The key lesson here is that enforcing security is a nuanced process that is not as simple as throwing in a few keywords into your code base. When designing your features, consider what it means for your users and customers to do so securely, in terms of usability, side effects, and of course, what it is you are truly trying to secure.
