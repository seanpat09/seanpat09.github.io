---
layout: post
title: The Joys of Dependency Injection Part 2 - Making a Mockery
date: '2017-04-08T08:51:00.000-07:00'
author: Sean Cuevo
tags: 
tags:
- interface
- tests
- StubAPI
- salesforce
modified_time: '2017-04-08T08:51:00.000-07:00'
---

In my [last post](/2017/04/08/interfaces.html) I discussed interfaces and how they can be used for dependency injection in Salesforce, allowing for flexible implementations of features. However, I can probably count on my hand the number of times I've actually had to use interfaces for this purpose. Designing to an interface assumes you are able to predict if and how something will be reused or repurposed. Instead, interfaces are particularly useful for mocking functionality during unit tests.

<!--break-->

Most of the testing I see is integration testing, also known as black box testing. Trigger tests usually fall into this category: when I insert object X, I expect Y to happen. For example:

```
trigger AccountTrigger on Account (before insert) {
    AccountTriggerBeforeInsertHandler.handle(Trigger.new);
}

public class AccountTriggerBeforeHandler{
    public void handle( List<Account> accounts ){
        AccountAssigner assigner = new AccountAssigner();
        assigner.assignOwner( accounts );
    }
}

@isTest
private class AccountTriggerTest {
    @isTest private void beforeInsert(){
        User expectedAssignee = TestUtils.buildAssignedUserForState('Colorado'); //Some test fixture method that creates our expected user;

        Account newAccount = new Account(Name='Test Account', State__c = 'Colorado');

        Test.startTest();
            insert newAccount;
        Test.stopTest();

        newAccount = [SELECT OwnerId FROM Account WHERE Id = :newAccount.Id];
        System.assertEquals( expectedAssignee.Id, newAccount.OwnerId, 'The Owner should be set correctly' );
    }
}
```

The test above tests that the before insert handler sets the Owner of an Account is set correctly. You could test other scenarios, but the minimum integration test here is that when you insert an Account, the Account gets assigned to someone. 

Unit tests, on the other hand, are a little more specific. What constitutes a unit is up for debate, but I consider it a single public method. If it is composed of several private methods, I still consider it a single unit. In our example, the `AccountAssigner` class would have its own set of tests that test all the scenarios for the `assignOwner` method.

But what about the `handle` method in `AccountTriggerBeforeHandler`? The integration test that fires the trigger will give us the coverage we need, but shouldn't we test the handler itself? And what would that test even look like? Unit tests should test what the method does. At first glance it seems that `handle` assigns owners to Accounts records. However, the method actually just *calls* the `assignOwner` method, it doesn't actually do the assigning. So the test should make sure that our method does just that. For example:

```
@isTest
private class AccountTriggerBeforeHandlerTest {
    @isTest private void handle(){
        User expectedAssignee = TestUtils.buildAssignedUserForState('Colorado'); //Some test fixture method that creates our expected user;

        Account newAccount = new Account(Name='Test Account', State__c = 'Colorado');

        Test.startTest();
            new AccountTriggerBeforeHandler().handle(new List<Account>{ newAccount } );
        Test.stopTest();

        System.assertEquals( expectedAssignee.Id, newAccount.OwnerId, 'The Owner should be set correctly' );
    }
}
```

This test is so similar to the integration test that we might as well not have it all. But how else can we test it? That's where interfaces come. We'll create an interface for the AccountAssigner object, with a special method to replace or constructor.

```
public class AccountAssigner implements IAccountAssigner{
    
    public interface IAccountAssigner{
        void assignOwner( List<Account> accounts );
    }
    
    @TestVisible static IAccountAssigner mock;
    public static construct(){
        if( Test.isRunningTest() && mock != NULL ){
            return mock;
        } else{
            return new AccountAssigner();
        }
    }

    public void assignOwner( List<Account> accounts ){
        //logic for assigning an owner
    }
}
```

I usually keep the interface within the class; it reduces the number of files you have and it's only purpose is to mock this class. However, feel free to put it in another file you want. I also added a `construct` method that returns a mock if we want and only if it's a test. Now that we have an interface, instead of instantiating `AccountAssigner` in the `handle` method, we 'll use the new `construct` method, allowing us to inject our mock during tests:

```
public class AccountTriggerBeforeHandler{
    public void handle( List<Account> accounts ){
        AccountAssigner.IAccountAssigner assigner = AccountAssigner.construct(); 
        assigner.assignOwner( accounts );
    }
}
```

We're now set up our to mock `AccountAssigner` in our test:

```
@isTest
private class AccountTriggerBeforeHandlerTest {
    public class MockAssigner implements AccountAssigner.IAccountAssigner{
        Boolean assignOwnerCalled = false;
        public void assignOwner(List<Account> accounts){
            assignOwnerCalled = true;   
        }
    }
    
    @isTest private void handle(){
        MockAssigner mock = new MockAssigner();
        AccountAssigner.mock = mock;

        Account newAccount = new Account(Name='Test Account', State__c = 'Colorado');

        Test.startTest();
            new AccountTriggerBeforeHandler().handle( new List<Account>{ newAccount } );
        Test.stopTest();

        System.assert( mock.assignOwnerCalled, 'The assignOwner method should have been called' );
    }
}
```

First we create `MockAssigner`, a class that implements the `IAccountAssigner` interface. All the mock will do is mark that the method has been called, which is all that we care about in this test. The test method will then instantiate a mock and inject it into the `mock` variable in `AccountAssigner`, which will then in turn be used by the `construct` method.

We now have an adequate unit test for our `handle` method; we programatically know that it calls `assignOwner`. We also decouple the functionality of `AccountTriggerBeforeHandler` from `AccountAssigner`. Should we need to change the logic within `assignOwner`, the unit test for `handle` would not have to change. We could also test what happens if `assignOwner` threw an exception, hit governor limits, etc. because the test has complete control over what happens in the method. This gives us a lot of flexibility to write very comprehensive tests. As a side effect, programming this way also forces you to build in a more modular fashion.

I've been doing this for a few years now and it works nicely, but it also means a lot of extra code in production: interfaces, new contstructors, etc. In the Winter '17 release the Apex Stub API was announced. In the final part of this series, we'll discuss how to use the Stub API and how it compares to using interfaces for mocking.