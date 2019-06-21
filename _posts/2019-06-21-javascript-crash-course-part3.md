---
layout: post
title: A JavaScript Crash Course from a Apex Dev's Point of View - Part 3
date: '2019-06-21-T00:00:00.000-07:00'
author: Sean Cuevo
tags:
- salesforce
- lwc
- apex
- javascript
---

*At TrailheaDX '19, I was fortunate enough to do a talk on learning JavaScript fundamentals from an Apex developer's point of view. This series covers the topics from that talk*

We focused on first class functions in the <a href="{{page.previous.url}}">last post</a>, which should help set the stage for demystifying the `this` keyword.

## What's this?

In Apex, the `this` keyword is fairly straightforward: it refers to the current instance of an Object. Take this `Person` class as an example:

```
public class Person {
    String name;

    public String getName() {
        return this.name;
    }
}
```

When you instantiate a `Person` object, the method `getName` uses `this.name` to return that instance's value for the property `name`.

If you treat the `this` keyword in JavaScript similarly you will quickly run into some unexpected behavior. Remember, functions in JavaScript are first class. So while a function may be the value of a property in an object, it doesn't necessarily *belong* to that object. For example, consider this object:

```
let person = {
    name : 'Sean',
    getName : function() {
        return this.name;
    }
}
```

The `person` JavaScript object has a property called `getName` and it would seem that if you called that function, it should return the `name` property on that object (i.e. the value `Sean`). Depending on how the function was called, that would be true. But what if you assigned the value of `getName` to another variable, like so?

```
let nameGetter = person.getName;
```

Now the variable `nameGetter` has been essentially been removed from the object `person`. What does the `this` keyword in `nameGetter` refer to now? If you think of JavaScript like Apex, you would be tempted to say the `this` keyword still refers to the `person` object, but that's not the case.

Generally, `this` keyword is often said to be bound to the "call site", meaning that its values depends on the context in which the function is used. Just by looking at a function that contains the `this` keyword is not enough to tell you what it is bound to. It is not necessarily the object it is used in, but it can be!

This really frustrated me, because I wanted the `this` keyword to work like it does in Apex. But if you can let go of that notion of what you think it should be, you can than embrace it for what it can do. To do that, let's walk through the rules of how the `this` keyword is bound.

(Credit for the `this` hierarchy that follows goes to [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md), an amazing book that really helped me understand JavaScript more!)

## Default Binding

Let's start with "default binding", which is basically what `this` falls back on when all of the other rules that follow don't apply. In short, the default binding is the global scope. For example:

```
function identify() {
    console.log(this.name);
}

var name = 'Sean'

identify(); //Outputs 'Sean'
```

The variable `name` is in the global scope. So when you call the `identify` function, none of the other rules for binding the `this` keyword apply (you'll understand why later as you read on), so it is bound to the global scope and thus the log the string `Sean`. 

However, when `strict mode` is in use, which it will be in LWC, then the global scope is not eligible to be bound to `this`, so you will get an error instead.

```
function identify() {
    'use strict`
    console.log(this.name);
}

var name = 'Sean'

identify(); //TypeError: Cannot read property 'name' of undefined
```

## "new" Binding

Let's start at the top of the `this` binding hierarchy. When you call a function with the `new` keyword, you are invoking a constructor call of that function. When you call a function with the `new` keyword, that function behaves differently:

* The function creates an empty object
* Any references to the `this` keyword in the function will now refer to that object during it's execution
* At the end of the function, unless something is explicitly returned by the function, that newly created object will be returned (even if you don't have a return statement in your function)

Consider the following function named `Person`. It has a capital "P", but that does not affect it's behavior as a constructor. Instead, it typically signals that it is a constructor function that can and should be called with the `new` keyword, but using a lowercase `p` would work as well.

```
function Person(name) {
    this.name = name;
}
```

When we call this function with the new keyword, we can assign the object it creates to a variable.

```
let programmerSean = new Person('Sean');
console.log(programmerSean.name) //Outputs 'Sean'
```

Because we used the `new` keyword to invoke the `Person` function, the variable `programmerSean` is an object with a property `name`, that contains the string value `Sean`. What if we don't use the `new` keyword?

```
let programmerJoe = new Person('Joe');
console.log(programmerJoe.name) //TypeError: Cannot read property 'name' of undefined.
console.log(name) //Outputs 'Joe' because the `this` keyword fell back on the global scope!
```

Without the `new` keyword the `Person` function did NOT return an Object, so the variable `programmerJoe` is actually undefined. Also the `this` keyword could not use new binding, so it defaulted to the global scope. As a result, there is now a `name` property on the global scope that holds the string value `Joe`!

## Explicit Binding

In JavaScript, you can also explicitly set what you want the `this` keyword to be bound to using the functions `call`, `apply` and `bind`.

Consider the following function and object:

```
function greet(greetings) {
    console.log(greetings, this.name);
}

let sean = { name : 'Sean' }
```

The `call` function can be called on a function and accepts one to many parameters. The first parameter is the object that you want `this` to be bound to. And rest of the parameters are the parameters that you want to pass the function you are invoking. Using `call` on a function immediately invokes it. For example:

```
greet.call(sean, 'Hello', 'My Friend') //Outputs 'Hello My Friend Sean'
```

The `apply` function works the same, but accepts only 2 parameters. The first parameter is the same, it's the object you want `this` to be bound to. The second parameter, however, is an array that holds all of the parameters you want to pass the function you are invoking. For example:

```
greet.call(sean, ['So Long', 'Farewell']) //Outputs 'So Long Farewell Sean'
```

The `bind` function works differently in that it does not invoke the function. When you call `bind` on a function, you pass it one parameter, which will be the object you want the `this` keyword bound to. `bind` then returns that function with `this` already bound, which can you assign to a variable. For example:

```
let boundFunction = greet.bind(sean);
boundFunction('Good Morning'); //Outputs 'Good Morning Sean'
```

In short, use `call` and `apply` when you want to invoke a function with a specified `this` keyword and use `bind` when you want to specify the `this` keyword, but don't want to invoke the function until later.

## Implicit Binding
When a object has a function as a property, when you call that function from that object, the `this` keyword is bound to that bound. This is known as implicit binding. Here is the `person` object that we looked at earlier that contained a function as a property:

```
let person = {
    name : 'Sean',
    getName : function() {
        return this.name;
    }
}
```

When you call `person.getName()`, due to implicit binding the `this` keyword refers to the `person` object and thus returns the string value `Sean`. As we illustrated earlier, you can also lose this implicit binding if you call the function outside of the context of the object.

```
let nameGetter = person.getName;
nameGetter(); //returns nothing
```

You will often run into this in LWC. Consider this example component:

```
import { LightningElement, track } from 'lwc'
export default class JobTracker extends LightningElement {
    @track
    jobRunning = false;

    handleClick() {
        this.jobRunning = true
    }
}
```

At first class, it looks like JavaScript class are treating the `this` keyword similarly to how it works in Apex. It looks like `this` is mean to represent that instance of the class, which is why the `handleClick` function can access the `jobRunning` property. In actuality, this only works because of implicit binding. Let's rewrite this component to illustrate how the `this` keyword does not necessarily refer to the class.

```
import { LightningElement, track } from 'lwc'
export default class JobTracker extends LightningElement {
    @track
    jobRunning = false;

    handleClick() {
        markJobRunning();

        function markJobRunning() {
            this.jobRunning = true
        }
    }
}
```

We have now created a nested function called `markJobRunning` within the `handleClick` function. `markJobRunning` is not called with the `new` keyword, so new binding is not in effect. Explicit binding is also not being used and we can tell that implicit binding is not being used because `markJobRunning` is not being called from an object. So that means that default binding is in effect. Of course, since LWC runs in strict mode, that means `this` is going to undefined, so we'll get an error saying `Cannot read property jobRunning of undefined`.

## Binding Hierarchy

To review, use the following hierarchy to determine what `this` is bound to.

* Is the function being called with `new`?
    * The `this` keyword refers to the "new" object constructed by the function
* If not, is the function being called with `call` or `apply`, or instantiated with `bind`?
    * The `this` keyword refers to the specified object
* If not, is the function being called off of an object?
    * The `this` keyword refers to the object the function was called off of
* If not, default binding is in effect
    * The `this` keyword refers to the global scope, or is undefined if strict mode is enforce.

## Arrow functions

There is of course an exception to all of this! Arrow functions are not just shorthand for writing functions. The `this` keyword actually behaves differently when you use an arrow function. For those who are unfamiliar here is the same function written with the `function` keyword and as arrow function:

```
let oldFunction = function(x) { console.log(x) }
let arrowFunction = x => console.log(x)
```

I love the shortened syntax, but if you have the `this` keyword in a function that means that two different syntaxes are not interchangeable. To illustrate, let's look at an example that demonstrates that problem that arrow functions attempt to solve. Consider our `Person` constructor function again.

```
function Person(name) {
    this.name = name;
    this.delayedGreet = function() {
        setTimeout(
            function() {
                console.log("Hi, my name is ', this.name);
            },
            100
        )
    }
}

let sean = new Person('Sean');
sean.delayedGreet() // Eventually outputs "Hi, my name is undefined"
```

So our `Person` function constructs an object with a `name` property and a `delayedGreet` property that calls `setTimeout` with a callback function that logs out a greeting using the `name` property of the `this` keyword. However, after calling the `Person` constructor, when we call `delayedGreet`, it outputs `Hi, my name is undefined` instead of the expected `Hi, my name is Sean`. What's happening here? What does the `this` keyword in `this.name` refer to?

At first glance you might think that new binding should be in effect. However when the `delayedGreet` property is created, the callback passed to the `setTimeout` function is not being invoked yet, so new binding IS NOT in effect. When that callback is eventually invoked, the `this` keyword is bound to however the `setTimeout` function binds it, so `this.name` ends up being undefined.

Arrow functions attempt to fix this confusion. Here's the same method written with an arrow function:

```
function Person(name) {
    this.name = name;
    this.delayedGreet = function() {
        setTimeout(
            () => console.log("Hi, my name is ', this.name),
            100
        )
    }
}

let sean = new Person('Sean');
sean.delayedGreet() // Eventually outputs "Hi, my name is Sean"
```

The same function but written with an arrow function and now it works! What's happening here? Well, arrow functions use "lexical scope" for defining the `this` keyword. In other words, whatever `this` is bound to when the function was instantiated is what `this` will be bound to. In our example, the `this` keyword will use new binding so that it is bound to the same object that was created during the constructor.

Are you confused? You should be! In my opinion, this "fix" just creates more mental overhead. It's another exception you have to keep track of when you already have to deal with a somewhat confusing hierarchy. It's important to know what's going on here because some developers do like this "solution" that arrow functions provide. I, however, instead recommend using explicit binding.

```
function Person(name) {
    this.name = name;
    this.delayedGreet = function() {
        setTimeout(
            function() {
                console.log("Hi, my name is ', this.name);
            }.bind(this), //Explicitly bind this to the "new" binding"
            100
        )
    }
}

let sean = new Person('Sean');
sean.delayedGreet() // Eventually outputs "Hi, my name is Sean"
```

By using the `bind` method, I am explicitly setting what I want the `this` keyword to be so that it is very clear as to what I intend. My advice? If you have to refer to the `this` keyword, avoid using an arrow function.

Hopefully you now have a better understanding of how the `this` keyword works, which can make reading JavaScript much less intimidating. In our next post, we'll wrap up with some debugging options that you have in JavaScript that you previously did not have in Apex.