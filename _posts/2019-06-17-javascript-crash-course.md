---
layout: post
title: A JavaScript Crash Course from a Apex Dev's Point of View - Part 1
date: '2019-06-17-T00:00:00.000-07:00'
author: Sean Cuevo
tags:
- salesforce
- lwc
- apex
- javascript
---

*At TrailheaDX '19, I was fortunate enough to do a talk on learning JavaScript fundamentals from an Apex developer's point of view. This series covers the topics from that talk*

I've been a developer for about 8 years now and while most of that time has been focused on working on the Salesforce platform, I've also had the opportunity to work on Golang, Ruby, and of course, JavaScript. Just like the languages we use to speak with each other, part of successfully using a programming language is to embrace the nuances that are commonly followed by other users of that language. In other words:

> Stop developing in JavaScript as if you were writing Apex.

During this series, we'll cover some topics that I believe provide a strong foundation to getting started with JavaScript by comparing its features to analogous features from Apex.

## JavaScript is a Dynamically Typed Language, Apex is Static

In a statically typed language like Apex, variables must be declared with their type and thus those variables can only reference their declared type. Here's a simple Apex method for example:

```
Integer increment(Integer x) {
    return x + 1;
}
```

Because of static typing, the method `increment` can only accept an Integer as a parameter and can only return an Integer. This provides a level of protection at compile time as there is a stricter contract as to what your method will accept and return, leaving you with less surprises at runtime.

In a dynamically typed language like JavaScript, variable types are defined at runtime. Here's the same method as a JavaScript function:

```
function increment(x) {
    return x + 1;
}
```

The parameter `x` can be anything! A string, an integer, or some kind of arbitrary object. You also do not necessarily know what it will return just by looking it. You lose that compile time protection, but you also get more flexibility. For instance, dynamic typing makes defining arbitrary data easier. Here's a function that returns an arbitrary data structure that is declared inline:

```
function returnData() {
    return {
        a : 'some',
        b : 'random',
        c : {
            d : 'nested data
        }
    }
}
```

I can't count the number of times while writing Apex where I just need to process some data and return it in some random structure. Here is the equivalent of that data structure in Apex:

```
class RandomData {
    String a;
    String b;
    NestedData c;
}

class NestedData {
    String d;
}

class RandomData returnData() {
    RandomData rand = new RandomData();
    NestedData nest = new NestedData();

    rand.a = 'some';
    rand.b = 'random';
    rand.c = nest;
    nest.d = 'nested data';

    return rand;
}
```

Because Apex is statically typed, you would need to define objects that follow the data structure you want. You get some type safety here but if this was just some private helper method in a class, that type safety is probably outweighed by the extra code that you have to write (and thus maintain!) This is a pretty contrived example, but you have probably seen this when making wrapper classes for SObjects when you wanted to encapsulate an SObject in addition to other data that does not necessarily exist as a field on that SObject. For example:

```
class OpportunityWrapper() {
    Opportunity opp;
    Boolean shouldDisplay;
}
```

Sure that's only four lines, but where should those four lines live? In the controller it is used in? In its own file? What if you have to add more properties? What if those properties aren't necessarily related? You can see how you can start to spiral out of control, all over some data structure.

The increased flexibility of dynamic typing, however, can lead to some unexpected results. Consider this `concatenate` function that accepts two parameters and logs their concatenation as a string.

```
function concatenate(a, b){
    console.log(a + b);
}
```

If you called function with the strings "hello" and "world", you'll see the expected output "helloworld" with no spaces. If you called function with the integers "1" and "2", you would expect the output "12", but it would actually output "3". The function just adds the parameters together and adding two integers together behaves differently from adding two strings together. To fix this, add an empty string to the concatenation so the parameters are coerced into strings. Here's the fixed version of function:

```
function concatenate(a, b){
    console.log(a + '' + b);
}
```

Now passing the integers "1" and "2" to the function will output the expected "12". So when you are writing JavaScript, make sure to be more defensive when handling variable types to prevent unexpected results.

## Scoping

In short, scope refers to visibility of variables within your code. JavaScript and Apex have some differences when handling scope.

Apex is pretty straightforward with **block level** scope. You can think of a block as anything between a pair of curly braces. Scope flows outward in Apex. When a variable is used in a block, the compiler searches within that block for its definition. If the variable is not defined in there, the compiler searches outside of that block to see if it is a static or instance property in that class. If it still can find it, then it checks the global scope (which contains the definition for tokens like Test and Schema). If that fail, then you get a compilation error.

This outward flow of scope also explains why in Apex you are able to have some level of duplicate variable names without issue. For example, in this Apex method, the variable `i` is defined twice: once within the `blockScope` method and once as an instance variable. The `blockScope` method uses the `i` that is declared within the block of the method as opposed to the instance variable defined outside of the scope.

```
private Integer i = 0;

void blockScope() {
    Integer i = 10;
    System.debug(i);
}

blockScope(); //outputs 10;
```

In JavaScript, scope depends on how you declare your variables. You are probably most familiar with using the `var` keyword to declare variables, which provide **function level** scope.

For example, consider this function where the variable `greeting` is defined within an `if` block. The `greeting` variable is still visible outside of the block due to the function level scoping.

```
function greet() {
    if(true) {
        var greeting = 'hello!';
    }

    console.log(greeting) //Outputs 'hello!'
}
```

If you forget to use the `var` keyword, JavaScript will actually put that variable on global scope (i.e. the `window` object).

```
function greet() {
    if(true) {
        greeting = 'hello!';
    }

    console.log(greeting) //Outputs 'hello!'
}
```

Fortunately, if you write you code with `use strict`, then strict mode will prevent this from happening and will throw an error

```
function greet() {
    'use strict'
    if(true) {
        greeting = 'hello!'; //Throws "ReferenceError: greeting is not defined in scope"
    }

    console.log(greeting) 
}
```

Lightning Locker actually enforces strict mode everywhere so you don't have to specify `use strict`, but it is important to understand the mechanism behind this scoping.

ES6 introduced the two new keywords `let` and `const` for variable declaration which provide block level scope like in Apex. `let` is used for variables that you want to be able to reassign values to while `const` variable values cannot be reassigned. In the following function, the variables `greeting1` and `greeting2` are not accessible outside of the `if` block;

```
function greet() {
    'use strict'
    if(true) {
        let greeting1 = 'hello';
        const greeting2 = 'world!';
    }

    console.log(greeting1) //Throws "Uncaught ReferenceError: greeting1 is not defined"
    console.log(greeting2) //Throws "Uncaught ReferenceError: greeting2 is not defined"
}
```

In general, I tend to default to `const` if the variable won't be reassigned, then I use `let` if I need to change the variable and only using `var` when I need function level scope, though I don't run into many scenarios where that is necessary.


These are some very basic differences between JavaScript and Apex. In the next part, we'll dive deeper by covering first class functions and how that changes the way you approach writing code in JavaScript compared to Apex.