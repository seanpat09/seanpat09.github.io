---
layout: post
title: Lightning Web Components - First Impressions
date: '2018-12-26-T00:51:00.000-07:00'
author: Sean Cuevo
tags:
- salesforce
- lightning web components
- lightning
- lwc
---

I am not a huge of fan of Aura, at least conceptually. As a tool it is fine, but a JavaScript framework seemed like a good opportunity to bridge the gap between Salesforce and non-Salesforce web development. Instead, we got Aura which may as well have been another Salesforce specific language that you would have to learn in order the break into the already niche world of Salesforce development. So when they announced [Lightning Web Components (LWC)](https://developer.salesforce.com/blogs/2018/12/introducing-lightning-web-components.html) earlier this month I was skeptical. JavaScript fatigue is bad enough - Salesforce specific JavaScript fatigue seems almost comical.

However, the blog post seemed to indicate that they feel the same way:

>Frameworks became the language. React, Angular, and the Lightning Component Framework are all JavaScript frameworks, but they provide such a high level of abstraction that they feel like different languages. As a result, skills were not transferable, and developers were hard to find and ramp up.

And their solution:

> Lightning Web Components is the Salesforce implementation of that new breed of lightweight frameworks built on web standards. It leverages custom elements, templates, shadow DOM, decorators, modules, and other new language constructs available in ECMAScript 7 and beyond.

A standards based approach was intriguing to me, so I wanted to give LWC a spin and share what I learned along the way.

## Dynamic Multi Line Editable Table
Something I have built over and over again is a mechanism to input multiple lines of input and then submit it to the server for whatever processing. I wanted to be able to feed a component an array of columns and it dynamically builds my table for me. I wanted to able to code something like this:

```
<c-multi-edit-table 
    column-list='\[{ "label" : "Name", "apiName" : "Name"} , { "label" : "Email", "apiName" : "Email" }, { "label" : "Phone", "apiName" : "Phone" }]'
    title='Attendees'>
</c-multi-edit-table>

<c-multi-edit-table 
    column-list='\[{ "label" : "Fee Name", "apiName" : "FeeName" }, { "label" : "Amount", "apiName" : "Amount" }]'
    title='Estimated Fees'>
</c-multi-edit-table>
```
And get this:
![dynamic table screenshot](/assets/img/DynamicTable.PNG)

For those of you playing along at home, here are some resources to help you get started:

[Quick Start: Lightning Web Components](https://trailhead.salesforce.com/en/content/learn/projects/quick-start-lightning-web-components)

[LWC Documentation](https://developer.salesforce.com/docs/component-library/documentation/lwc)

[GitHub repo with project code](https://github.com/seanpat09/dynamicFieldsLWC)

## Lesson #1: Naming Matters

I actually learned this last, but I thought it was very important to point out and save someone an hour trying to figure out what's wrong.

A component is composed of three parts: HTML, JS, and CSS

```
multiEditTable
|_ multiEditTable.html
|_ multiEditTable.js
|_ multiEditTable.css
```

Which you can then reference in other components by using

```
<c-multi-edit-table></c-multi-edit-table>
```

Turns out that the naming is very important here. The camel casing is parsed and each word is separated with a `-`. This causes an issue if you start your component with a capital letter like I originally did (i.e. MultiEditTable). In order to reference it you'll need to use:

```
<c--multi-edit-table></c--multi-edit-table> <!-- notice the two dashes before "multi" -->
```

The hard part about working with a pre-release is not knowing what is a bug and what is working as intended. My guess is that the parser is putting a dash before every capital letter, but the autocomplete in the VS Code plugin didn't pick this up so I spent an hour or so trying to figure out why my component would not render. The only error message I would get is `Compilation Failed`. I doubt this is the way it is intended to work, so to be safe just use a lower case letter when naming your components.

## Lesson #2: Computed Keys Still Not Allowed

One obstacle I always ran into while doing something dynamic like this was working with computed keys within the component markup. Here's something you can't do in Aura:

```
<aura:attribute name="record" type="Object[]"/> 
<aura:iteration items="{!v.columns}" var="column">
    <lightning:input label="{!v.column}" value="{!v.record[column]}"/>
</aura:iteration>
```

Aura won't let you bind computed key values (i.e. `value="{!v.record[column]}"` is invalid). To get around this, I had to create 3 separate components - 1 for the table itself, 1 for each row, and one for each input column within each row. It creates so much code and the components are so tightly coupled that you would never use them separtely. Here's a basic version of the end product in aura:

```
-- Table component
<aura:component>
    <aura:attribute name="record" type="Object[]"/> 
    <aura:attribute name="columns" type="String[]"/>
    <aura:iteration items="{!v.records}" var="record">
        <c:InputRow row="{!v.record}" columsn="{!v.columns}"/>
    </aura:iteration>
</aura:component>

-- InputRow Component
<aura:component>
    <aura:attribute name="row" type="Object"/> 
    <aura:attribute name="columns" type="String[]"/>
    <aura:iteration items="{!v.columns}" var="column">
        <c:InputCell record="{!v.row}" field="{!v.column}"/>
    </aura:iteration>
</aura:component>

-- InputCell Component
<aura:component>
    <aura:attribute name="record" type="Object"/> 
    <aura:attribute name="field" type="String"/>
    <aura:attribute name="value" type="String"/>
    <lightning:input value="{!v.value}" onchange="{!c.updateRecord}"> --updateRecord would set the field value on record
</aura:component>
```

If you could use computed keys, this would actually be fairly straightfoward. Here's what I tried first using LWC:

```
<template for:each={record} for:item="record">
    <template for:each={columns} for:item="column">
        <td key={column}>
            <lightning-input name={column} value={record[column]}></lightning-input>
        </td>
    </template>
</template>
```

Unfortunately this still does not work and the compiler explicitly states that you cannot use computed keys. I was so disappointed that I almost stopped there, but I figured I had gotten this far and I might as well see if LWC would provide other benefits.

## Lesson #3: One-Way vs. Two-Way Binding

When I built this table using Aura, I created an array of Objects in the top most component and passed the individual Objects in that array down the hierarchy.

```
<aura:attribute name="records" type="Object[]"/> 
<aura:iteration items="{!v.records}" var="record">
    <c:InputRow row="record">
</aura:iteration>
```

In Aura, you can define an attribute as having one way or two binding using `{#v.row}` and `{!v.row}` respectively. Two way binding made keep tracking of changes in the rows easy. You can pass an Object all the way down the hierarchy of components and any changes to that Object are automatically propagated back to the parent for you.

With LWC, you only get one-way binding; there's no option for two-way binding. At first I was frustrated by this because if I wanted to mimic the two-way binding, I thought I would have to deal with broadcasting and handling events. However, the more I thought about it, the more I realized that enforcing one-way binding would lead to less side effects. Components now have to explicitly change their properties instead of being them changed by another component due to a shared reference to an object, which will likely be tracked using a different variable name. In the previous example the component tracked the objects using the variable `record`, whereas the `InputRow` component used `row`. When working with more complex components keeping track of these variable pairings in your head could make debugging tricky.

## Lesson #4: Standard Query Selectors
That aside, I still needed to track the changes made between each row. At this point, I didn't see any of the productivity increases that I was hoping for - I still needed to create multiple levels of components and now I needed build an event so I could capture the row changes. Then I realized that I was still thinking with an Aura mindset.

I started to wonder how I would solve this if I was just building this with vanilla JavaScript. I would probably just use `document.querySelectorAll('tr')` and just iterate over each input within each row and build an array of objects from there. In LWC, you can do just that. For example, here's how you can get the inputs from each row:

```
retrieveRecords() {
    var records = [];
    Array.from( this.template.querySelectorAll("tr.inputRows") ).forEach(row => {
        var record = {};
        Array.from( row.querySelectorAll("input") ).forEach(input => {
            record[input.name] = input.value;
        });
        records.push(record);
    })

    return records;
}
```

This is where LWC started to click for me. With LWC, I could think with a more standard mindset instead of trying to solve Salesforce specific problems with Salesforce specific tools. More importantly, I could search for my JavaScript questions on Google without having to prepend every query with `salesforce`.

## Lesson #5: More Intuitive Server Calls
Now that I have all the data I need, I needed to submit it to my Apex controller to do whatever processing (i.e. a database insert). This was much clearer in LWC.

In Aura:

```
submit : function(component, event, helper) {
    var action = component.get("c.submitRecords");
    action.setParams({
        "records": JSON.stringify(component.find("inputTable").get("v.recordList")),
    });

    action.setCallback(this, function(response) {
        var state = response.getState();
        if (state === "SUCCESS") {
            alert('Success!');
        } else if (state === "ERROR") {
            alert('Something went wrong...');
        }
    });

    $A.enqueueAction(action);
}
```

And the same method in LWC:

```
import { LightningElement } from 'lwc';
import submitRecords from '@salesforce/apex/MultiEditTableController.submitRecords';

export default class DemoComposition extends LightningElement {
    submit() {
        var allRecords = this.template.querySelector("c-multi-edit-table").retrieveRecords();
        submitRecords({ records: allRecords })
            .then(() => {
                alert('Success!');
            })
            .catch(() => {
                alert('Something went wrong...');
            })
    }
}
```

I don't want to argue style or readability, but what I really like about LWC is that it is very clear where everything is coming from. The `submitRecords` controller method is explicitly referenced in LWC so you know which Apex class it's referencing. Also because of the way LWC uses `import` to bring in Apex methods, you could in theory include methods from other Apex classes fairly easily. In Aura, you'd have to create a separate service component that hooked up to another Apex controller and then include that commponent for the sole purpose of accessing those methods.

## Next Steps:

After a few hours I was successfully able to create my table. This came as a surprise because we're still in the pre-release stage and there's not much documentation out there. LWC's adherence to web standards made this less of an issue because a lot of the problems I ran into weren't all Salesforce specific problems. I could not say the same about my initial experience with Aura.

However, the biggest surprise was something I haven't touched upon yet. In the next post, we'll cover testing LWC using Jest, all from the comfort of your local machine.