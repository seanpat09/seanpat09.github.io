---
layout: post
title: "Design Patterns in Salesforce: Abstract Factory"
date: '2022-05-12-T00:00:00.000-08:00'
author: Sean Cuevo
description: Using the abstract factory design pattern in Salesforce apex
image: /assets/img/building-blocks.jpg

tags:
- design patterns
- apex
- abstract factory
---

*Design patterns are powerful tool to build reusuable code, but they are hard to implement in without examples and use cases. This is a series that follows the book Design Patterns: Elements of Resuable Object-Oriented Software a provides examples that relate to Salesforce*

<figure>
  <img src="{{site.url}}/assets/img/building-blocks.jpg" alt=""/>
  <figcaption>
    Photo by <a href="https://unsplash.com/@ryanquintal?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Ryan Quintal</a> on <a href="https://unsplash.com/s/photos/building-blocks?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a>
  </figcaption>
</figure>

## Abstract Factory

The Abstract Factory design pattern is useful for creating related objects that follow an interface. I find this pattern particularly useful for building resuable Lightning Web Components. Let's use this pattern to build a resuable custom related list.

## Use Case: Viewing grandchild records on a detail page.

Out of the box related lists in Salesforce are great for displaying child records on a detail page, but what if you wanted to display all the grand child records? Imagine a national charity that oversees local branches. Each branch is represented as an Account record, and the volunteers for that branch are represented as Contact records. When a volunteer is onboarded, there are several onboarding tasks required before they can start any work, such as a background check, filling out emergency contact info, and creating a user in their Salesforce Community.

Region managers oversee local branches in their assigned region, so in addition to other information on the Account, the Region Managers want to see all the outstanding tasks for all the Contacts on the Account detail page in a table (for the sake of this post let's pretend this is the best solution).

## A non-resuauble approach

If you didn't care about resuability (which is ok if you don't have that bandwidth!), you could build an LWC like this (I can't guarantee this compiles)

```
//contactTasks.html -->

<template>
  <lightning-datatable
    key-field="id"
    data={data}
    columns={columns}>
  </lightning-datatable>
</template>

//contactTasks.js

import { LightningElement } from 'lwc';
import getTasks from '@salesforce/apex/ContactTasksController.getTasks';

const columns = [
  { label: 'Subject', fieldName: 'Subject' },
  { label: 'Assigned To', fieldName: 'WhoId'}
];

export default class ContactTasks extends LightningElement {
  data = [];
  columns = columns;
  recordId;

  async connectedCallback() {
      const data = await getTasks({ accountId: this.recordId });
      this.data = data;
  }
}

//ContactTasksController

public class ContactTasksController {
  @AuraEnabled
  public static List<Task> getTasks(Id accountId) {
    List<Task> allTasks = new List<Task>();
    for(Contact aContact : [SELECT (SELECT Subject, WhoId FROM Tasks) FROM Contact WHERE AccountId = :accountId]) {
      allTasks.addAll(aContact.Tasks);
    }

    return allTasks;
  }
}
```

A solution like this is fine, but in the future you might get a similar request where you want to display all tasks for all the Contacts related to an Opportunity. Or maybe just some other data. You could create a separate LWC with a separate Apex controller, but you might notice that the LWC isn't doing all that much, just displaying data. Most of the differences is in building the columns and getting the data. Wouldn't it be nice to *abstract* out the logic that builds the table data? Like some kind of *abstract factory*?

## The Abstract Factory Pattern

The abstract factory pattern can be broken down into 3 parts:

* A interface that specifies what an implementation must be able to build.
* A consumer that can request a specific implementation, but can handle any implementation of the interface
* Concrete implementations that fulfill the interface

So in this case we need an interface that can build the columns and rows for a table, a component that can specify which columns and tables to display and actually display them, and then 1 or more concrete implementations with the business logic. How would this look?

First let's create some objects to define what we're building:

```
public class CustomListColumn {
  public String label;
  public String fieldName;

  public CustomListColumn(String label, Object fieldName) {
    this.label = label;
    this.fieldName = fieldName;
  }
}

public class CustomListRow {
  public Object value;

  public CustomListRow(Object value) {
    this.value = value;
  }
}
```

Next our interface for our factory.

```
public interface ICustomListFactory {
  List<CustomListColumn> buildColumns();
  List<CustomListRow> buildRows(Id parentId);
}
```

And then our concrete implementation of that factory

```
public class AccountContactsTaskListFactory implements ICustomListFactory{
  public List<CustomListColumn> buildColumns() {
    List<CustomListColumn> columns = new List<CustomListColumn>();
    columns.add(
      new CustomListColumn('Subject', 'Subject'),
      new CustomListColumn('Assigned To', 'WhoId')
    );
  }
  public List<CustomListRow> buildRows(Id parentId) {
    List<CustomListRow> allTasks = new List<CustomListRow>();
    for(Contact aContact : [SELECT (SELECT Subject, WhoId FROM Tasks) FROM Contact WHERE AccountId = :parentId]) {
      for(Task aTask: aContact.Tasks) {
        allTasks.add(new CustomListRow(aTask));
      }
    }
    return allTasks;
  }
}
```

Next let's make our component more generalized. It now gets the columns in addition to the rows from the Apex controller. It also passes a `listType` parameter to specify which implementation should be used
```
//customRelatedList.html -->

<template>
  <lightning-datatable
    key-field="id"
    data={data}
    columns={columns}>
  </lightning-datatable>
</template>

//contactTasks.js

import { LightningElement } from 'lwc';
import getTableData from '@salesforce/apex/CustomRelatedListController.getTableData';

export default class CustomRelatedList extends LightningElement {
  data = [];
  columns = columns;
  recordId;

  @api
  listType

  async connectedCallback() {
      const tableData = await getTableData({ recordId: this.recordId, listType: this.listType });
      this.data = tableData.rows;
      this.columns = tableData.columns;
  }
}
```

Finally we create a controller that will handle getting the correct implementation based on the listType passed from the LWC

```
public class CustomRelatedListController {
  @AuraEnabled
  public TableData getTableData(Id recordId, String listType) {
    ICustomListFactory factory = getCustomListFactory(listType);
    
    TableData data = new TableData();
    data.rows = factory.getRows(recordId);
    data.columns = factory.getColumns();

    return data;
  }

  private ICustomListFactory getCustomListFactory(String listType) {
    if(listType === 'AccountContactsTasks') {
      return new AccountContactsTaskListFactory();
    } else {
      throw CustomRelatedListControllerException('List Type: ' + listType + ' not supported');
    }
  }


  public TableData {
    @AuraEnabled
    List<CustomListColumn> columns;
    List<CustomListRow> rows;
  }

  public CustomRelatedListControllerException extends Exception {}
}
```

## Benefits

This seems like a lot of extra code and for just a single use case it probably is. The value comes with its reuse. Let's add another table, this time displaying Contact information related to an Opportunity. Specifically, the Contact's full name, parent account, grandparent Account (i.e. up an additional Account level in the hierarchy), and the Account website.

Here's the concrete implementation:

```
public class OpportunityContactListFactory implements ICustomListFactory{
  public List<CustomListColumn> buildColumns() {
    List<CustomListColumn> columns = new List<CustomListColumn>();
    columns.add(
      new CustomListColumn('Full Name', 'fullName'),
      new CustomListColumn('Parent Account', 'parentAccount'),
      new CustomListColumn('Grandparent Account', 'grandParentAccount'),
      new CustomListColumn('Website', 'website')
    );
  }
  public List<CustomListRow> buildRows(Id parentId) {
    List<CustomListRow> contacts = new List<CustomListRow>();
    for(OpportunityContactRole ocr :
      [ SELECT Contact.Fullname,
               Contact.Account.Name,
               Contact.Account.Account.Name,
               Contact.Account.Website
        FROM OpportunityContactRole WHERE OpportunitId = :parentId]
    ) {
        Map<String, String> contactRow = new Map<String, String>{
          'fullName' => ocr.Contact.FullName,
          'parentAccount' => ocr.Contact.Account.Name,
          'grandParentAccount' => ocr.Contact.Account.Account.Name,
          'website' => ocr.Contact.Account.Website,
        }
        contacts.add(contactRow);
    }
    return contacts;
  }
}
```

And then we update the controller to handle this new list type:

```
  private ICustomListFactory getCustomListFactory(String listType) {
    if(listType === 'AccountContactsTasks') {
      return new AccountContactsTaskListFactory();
    } else if(listType === 'OpportunityContacts') {
      return new OpportunityContactListFactory();
    } else {
      throw CustomRelatedListControllerException('List Type: ' + listType + ' not supported');
    }
  }
```

And now our LWC can handle a different use case given a different `listType` API value, without having to change any of its code! Arguably the `getCustomListFactory` method should be pulled into its own class so that the controller also does not have to be updated either.

I find myself using this pattern often with LWCs to take advantage of the reusability of components. It can take a little extra upfront work, but can save you a lot of time in the long run in terms of code you don't have to write and also for creating small testable components.
