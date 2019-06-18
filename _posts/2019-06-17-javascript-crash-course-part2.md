---
layout: post
title: A JavaScript Crash Course from a Apex Dev's Point of View - Part 2
date: '2019-06-18-T00:00:00.000-07:00'
author: Sean Cuevo
tags:
- salesforce
- lwc
- apex
- javascript
---

*At TrailheaDX '19, I was fortunate enough to do a talk on learning JavaScript fundamentals from an Apex developer's point of view. This series covers the topics from that talk*

In the <a href="{{page.previous.url}}">last post</a>, we covered some basic differences in JavaScript and Apex, focusing on dynamic typing and scope. In this post, we'll focus on first class functions in JavaScript, a feature that I believe is a fundamental difference in how you write code compared to Apex.

## First Class Functions

Unlike Apex, functions are "first class" in JavaScript. What does that mean? It means that functions can be:

* Passed as arguments to other functions, which are generally referred to as "callbacks"
* Used as return values from functions
* Assigned to variables
* Stored as attributes in data structures.

In Apex, methods can only be passed around when encapsulated in an object, and then called from that object.

Here is an example of first class functions in action:

```
function fetchAction() {
    return function() { console.log('you called?') }
}

function doTheThing(f) {
    f();
}

let anAction = fetchAction();
doTheThing(anAction); //Outputs: "you called?"
```

The first function `fetchAction` returns a simple function that logs the string `you called?`. The second function `doTheThing` accepts a callback and executes it. So we can call `fetchAction`, assign the return value to the variable `anAction`, and then pass that to `doTheThing`, which executes that action, logging the string `you called?`. As you can see, function values do NOT have the parenthesis. Adding the parenthesis to a function invokes the function.

You will often see this in LWC with imperative Apex, which imported as Promises. You can chain Promises with the `then` and `catch` functions, both of which accept a callback as parameters. When you call an Apex method in LWC, after the method completes, the `then` function executes the callback passed to it. If an exception is thrown, the `catch` method executes the callback passed to it.

For example:

```
import {LightningElement, track} from 'lwc';
import submitRecords from '@salesforce/apex/MyController.submitRecords';

export default class MyComponent extends LightningElement {
    @track allRecords
    
    submit() {
        submitRecords({ records: allRecords })
            .then(function(){
                alert('Success')
            })
            .catch(function(){
                alert('Something went wrong...')
            })
    }
}
```

When the `submit` function is called in our component, that executes the imperative Apex method `submitRecords`. If that method resolves successfully, the `then` function executes a callback function that alerts the string `Success`. If an exception is thrown, the `catch` function executes a callback function that alerts the string `Something went wrong`. Since functions are first class, we are able to declare the callbacks parameters as inline functions. But you could also declare the callbacks outside of the functions like so:

```
import {LightningElement, track} from 'lwc';
import submitRecords from '@salesforce/apex/MyController.submitRecords';

export default class MyComponent extends LightningElement {
    @track allRecords
    
    submit() {
        submitRecords({ records: allRecords })
            .then(handleSuccess)
            .catch(handleError)

        function handleSuccess(){
            alert('Success')
        }

        function handleError(){
            alert('Something went wrong...')
        }
    }
}
```

In this example, the behavior is the same, but this time the functions are declared outside of the parameters and then we pass `handleSuccess` and `handleError` to `then` and `catch` respectively. Remember, when referring to functions as values, do not add the parenthesis. `handleSuccess` is the function itself while `handleSuccess()` will call the function and pass the value of whatever is returned by that function.

Let's walk through a more practical example to help illustrate how this affects the way we write code. Given an array of contacts with their names and the group they each belong to, I want to get a concatenated string of all the contacts that belong the group "Avengers" so that I can print them on a document. Here is the list:

```
let contacts = [
    { groupName : 'Avengers',       name : 'Steve Rogers' },
    { groupName : 'Avengers',       name : 'Tony Stark' },
    { groupName : 'Avengers',       name : 'Natasha Romanov' },
    { groupName : 'Justice League', name : 'Diana Prince' },
    { groupName : 'Justice League', name : 'Bruce Wayne' },
    { groupName : 'X-Men',          name : 'Charles Xavier' },
]
```

If I were to build this in JavaScript with an Apex mindset, you might do something like this:

* Initialize a blank string
* Iterate through all the contacts
* If the contact belongs to a desired group, concatenate their name to the string

```
function concat(contacts) {
    let concatenatedNames = '';
    for (let i = 0; contacts.length; i++){
        if (contacts[i].groupName === 'Avengers') {
            concatenatedNames = concatenatedNames + '|' + contacts[i].nae;
        }
    }

    return concatenatedNames
}
```

There's nothing inherently wrong with this code; it does the job. But you typically wouldn't see it written like this. The `Array` object has functions that allow you to iterate over the items in an array while taking advantage fo first class functions.

* `filter` - iterates over each value in a collection and passes each item to a callback for processing
    * If the callback returns a "truthy" value, then the item will be added to a new array
    * Otherwise it does not add the item to the new array.
    * When complete, the new filtered array is returned
* `map` - iterates over each value in a collection and passes each item to a callback for processing.
    * Whatever is returned by that callback is added to new array
    * When complete, the new array is returned
* `reduce` - iterates over each value in a collection and passes each item and an aggregator to a callback for processing.
    * Whatever is returned by the callback becomes the "aggregator" object. This aggregator is passed to the next iteration of the callback
    * When complete, the aggregator is returned
    * Essentially, you are "reducing" an array into an object

Let's do the same concatenation, but this time leveraging these functions:

```
function concat(contacts) {
    return contacts
        .filter( function(contact) { return contact.groupName === 'Avengers' } )
        .map( function(contact) { return contact.Name }, [] )
        .reduce (function(concatenatedNames, name) { return concatenatedNames + '|' + name; }, '')
}
```

`filter` builds an array of contacts that are in the "Avengers". Since the return value of `filter` is an array, we can chain the `map` function to it, which will build an array of just the name strings from the contacts. `map` also returns an array, so we chain `reduce` to it, which will builds our concatenated string. To be more succinct, here's same method with arrow functions.

```
function concat(contacts) {
    return contacts
        .filter( c => c.groupName === 'Avengers' )
        .map( c => c.Name )
        .reduce( (concatenatedNames, n) => concatenatedNames + '|' + n, '')
}
```

This is a common pattern you'll see in JavaScript: call some function to get some data and use callbacks to process that data. Because functions are first class in JavaScript, execution tends to favor this pattern of passing around utility functions that are chained to produce an end result as opposed to using instance methods that manage an object's state.

I actually don't know if a `for` loop is faster or slower than using `filter`, `map` and `reduce`, but I do know that using `filter`, `map` and `reduce` is more prevalent. Remember, part of writing code is making it more readable to other developers. As the saying goes, when in JavaScript, do as the JavaScriptors do!

That covers the basics of first class functions and how they can change your approach to writing code in JavaScript vs. Apex. In the next post, we'll focus on demystifying the `this` keyword identifier and how it is drastically different from the `this` keyword in Apex.

