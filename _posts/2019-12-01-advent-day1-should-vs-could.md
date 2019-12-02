---
layout: post
title: So-Called Advent Calendar Day 1 - Static Interfaces
date: '2019-12-01-T00:00:00.000-07:00'
author: Sean Cuevo
description: Static methods in Apex can satisfy an interface, but that does not mean you should do it.
tags:
- programming
- Salesforce
- interface
---

*In the spirit of the holiday season, this is a series of short blog posts covering random things I have learned while doing Salesforce development, one for each day of Advent.*

<figure>
  <img src="{{site.url}}/assets/img/keepout.jpg" alt="Picture of Lisa Simpson looking at sign saying: Keep out. Or Enter. I'm a sign, not a cop"/>
    <figcaption>Sure you can do it, but is it a good idea?</figcaption>
</figure>

Some time ago I was working on a fix for a bug and as usual I began with trying to see if I could reproduce the bug in a unit test. I forget the exact nature of the bug, but I remember the method in question called out to a fairly complex service class that performed a lot of unrelated logic. Also, I remember that the service class wasn't really related to bug, but it would throw an exception if the test didn't include the appropriate setup.

Naturally, my first thought was to use the StubAPI or an interface to stub out the service class. Unfortunately the method was static, so that was not possible as it was currently written. I started considering creating an interface and wrapping the static methods in instance methods so I could mock the class when I realized that the class was already implementing an interface.

But all the methods in this class were static. And the interface it was implementing wasn't empty, it had several methods in it. As far as I knew, interface methods needed to be implemented using instance methods - you can't have a static interface method. Yet this class was compiling and had been for years. What's going on here? Can static methods satisfy an interface?

To test this, in my scratch org I defined a very simple interface:

```
public interface MyInterface {
    void doTheThing();
}
```

Then I tried implementing that interface using a static method:

```
public with sharing class StaticImplementation implements MyInterface {
    public static void doTheThing() {
        System.debug('I am static!');
    }
}
```

And it worked! The static method fulfilled the interface! It compiled! I thought to myself that this will surely throw some kind of runtime error if I tried to use this interface. But to my surprise it does not; the interface will actually call the static method:

```
MyInterface staticImp = new StaticImplementation();
staticImp.doTheThing(); //Outputs "I am static!"
```

And to confirm, this isn't just some weird Apex quirk that allows you to call static methods from an instance of a class. I tried doing the same without the interface, constructing the implementing class directly:

```
StaticImplementation staticImp = new StaticImplementation();
staticImp.doTheThing();

//Throws a compilation error: "Static method cannot be referenced from a non static context: void"
```

I can't find anything suggesting that this is expected behavior and I couldn't find anything suggesting this was a bug either. The first thing I thought was "Great! I found a trick that allows me to mock static methods in a test!" I started creating my mock class and my unit tests when I paused to remember a quote from one of the greatest movies of our time:

> Yeah, but your scientists were so preoccupied with whether or not they could, they didn't stop to think if they should. - Dr. Ian Malcolm, Jurassic Park

If there is anything I have learned in my career, being "clever" is generally a red flag. It is so much more important for the maintenance of your code that it is intuitive, that it makes sense to other developers (including your future self). How much time have you spent trying to fix "clever" code that made no sense, that left you guessing at the intent of the original author? This was exactly the kind of thing that could confuse someone else later on, or even worse, that could break in a future release since it leverage

To be honest, I don't actually see Salesforce fixing this issue - just delivering the change could potentially break a bunch of code in the wild that happens to take advantage of this (purposefully or not). But just because you can do something, doesn't mean you should. Next time you are feeling clever, take a second to consider what that cleverness might cost you later.