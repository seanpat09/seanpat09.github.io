---
layout: post
title: Apex Quirks - Interfaces
date: '2019-04-23-T00:00:00.000-07:00'
author: Sean Cuevo
tags:
- salesforce
- apex
- interfaces
- static methods
---

The best part about working with code that you didn't write is learning something new, like running into a new pattern or a feature that you didn't even know existed. For the most part, you'll think to yourself "Neat!" put it in your pocket for future use, and move on. Occasionally, however, you'll run into something that stops you in your tracks. I don't just mean confusing, spaghetti code. I'm talking about the times when you think, "Wait, that shouldn't even work."

I stumbled upon a piece of code that implemented an interface, yet only contained static methods. For example:

```
public interface MyInterface {
    void doTheThing();
}
```

```
public with sharing class StaticImplementation implements MyInterface {
    public static void doTheThing() {
        System.debug('I\'m static!');
    }
}
```

To my understanding, an interface is fulfilled using instance methods, like so:

```
public with sharing class InstanceImplementation implements MyInterface {
    public void doTheThing() {
        System.debug('I\'m an instance!');
    }
}

MyInterface instanceImp = new InstanceImplementation();
instanceImp.doTheThing(); //Outputs "I'm an instance!"
```

But `StaticImplementation` was compiling successfully, so it appears to be valid. At first I thought, "Can you actually execute static methods off of an instance of that class?". Let's try:

```
StaticImplementation staticImp = new StaticImplementation();
staticImp.doTheThing(); //Error: Static method cannot be referenced from a non static context
```

As I expected, you cannot. So at least I wasn't grossly mistaken on how static methods are called. But if the interface is being fulfilled by that class, what happens when I call that method using the interface declaration? I guessed we'd get a weird runtime error:

```
MyInterface staticImp = new StaticImplementation();
staticImp.doTheThing(); //Outputs "I'm static!"
```

WHAT?! The static method is being run as an instance method!

I'm 99% sure this is a bug in the Apex compiler that just happens to work. Otherwise I am very mistaken in my understanding of how interfaces work in object oriented languages.

So what does this mean? My immediate thought was "Maybe I can use this to mock static methods!" But a cautionary quote came to mind:

>_"Your scientists were so preoccupied with whether or not they could, they didn't stop to think if they should." - Ian Malcolm in Jurassic Park_

My advice is don't try to leverage this as a feature. At best, you'll confuse other developers, including yourself in 6 months, as to why this works. At worst, if this is actually an Apex bug and is fixed in an update, you'll find yourself locked into older Apex API versions until you refactor this "feature" out of your existing code.

As a developer, you should aim to write code that has clear intent and is human readable. Leveraging "hidden" features/bugs is a red flag because it will potentially confuse others down the road, will have little to no documentation, and may not even exist in the future. "Clever" code should not come at the expense of clean and clear code, especially in a scenario like this that gives you so little benefit.

Hopefully if you run into this problem, now you'll understand why it works, and why you should not do the same.


