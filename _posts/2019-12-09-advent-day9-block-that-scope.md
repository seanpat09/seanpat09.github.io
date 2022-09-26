---
layout: post
title: So-Called Advent Calendar Day 9 - Block that scope!
date: '2019-12-09-T00:00:00.000-07:00'
author: Sean Cuevo
description: Using blocks to limit scope

tags:
- apex
- salesforce
- blocks
- scope
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

When it comes to writing code, *scope* is used to describe how visible a piece of code is to the rest of your code. For example, if a variable is in scope, then it can be referenced. An easy way to define a scope is within curly braces. In Apex, I generally see curly braces used in a few contexts:

**To define methods and classes**

```
public class MyClass {
    private static Object scopedToClass;
    public class void myMethod() {
      Object scopedToMethod;
    }
}
```

**To scope if statements**

```
if (myCondition) {
    Object scopedToIf;
} else {
    Object scopedToElse
}
```

**To scope loops**

```
for(Object x : records) {
    Object scopedToLoop;
}
```

But recently I noticed that you can you just use curly braces to create scope anywhere. For example, if you have a large method and want to limit the scope of parts it, you just use curly braces to create that scope without creating another method.

```
public void myMethod() {
    Object inMethodScope;

    {
        Object inBlockScope;
    }

    //This would fail compilation because
    //inBlockScope is out of scope of the rest of the method!
    System.debug(inBlockScope);
}
```

I can't think of many times when you would use this where a separate method or a separate class would be more appropriate. Maybe if you were really trying to manage some heap issues and breaking things out into a separate method isn't really feasible? It seems a little farfetched, but it's always fun to find another tool in the toolbox, just in case.