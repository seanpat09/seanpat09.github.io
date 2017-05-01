---
layout: post
title: The Joys of Dependency Injection Part 1 - Interfaces
date: '2017-04-08T07:51:00.000-07:00'
author: Sean Cuevo
tags: 
tags:
- interface
- tests
- salesforce
- dependency injection
modified_time: '2017-04-08T07:51:00.000-07:00'
---

Dependency injection helps you make code that is easy to maintain and test by

* Allowing "units" of code to only be concerned with their own functionality
* Making dependencies easily swappable without altering dependent code
* Allowing you to mock/control dependencies during tests so that you isolate what you are testing


<!--break-->

In Salesforce, this is made possible via Interfaces. In short, an interface is a type that you declare that has methods it contains, but not their implementation. In order for an object to be of that type, it must implement the interface and all it's methods. Here's a classic example:

~~~
public interface Animal{
    String speak();
}
~~~

You'll notice that this is slightly different from a class. The method `speak` has no body. Instead it is declared like a variable. When looking at this you should think, "The type `Animal` has the method `speak`, which returns a `String`. To get use out of it, you need to have a class implement the interface.

~~~
public class Dog implements Animal{
    public String speak(){
        return 'Bark!'
    }
}
~~~

Now we have a `Dog` class that `implements` `Animal`. This tells the code that `Dog` is of type `Animal`. In order to do so, we had to declare that `Dog` implements the `Animal` interface and also flesh out the method `speak`. If you don't implement all of the methods in the interface, you'll get a compilation error:

~~~
Dog: Class must implement the public interface method:
 String speak() from Animal
~~~

Now let's use our Dog:

~~~
public class AnimalChorus{
    public static void sing(Animal a){
        system.debug(a.speak());
    }
}

//in Execute Anonymous
Dog poochy = new Dog();
AnimalChorus.sing(poochy);

//Debug output:
Bark!
~~~

The method `sing` in `AnimalChorus` accepts objects of the type `Animal`. Since `Dog` implements `Animal`, it is accepted as a parameter for the method `sing`. However, because `sing` accepts an `Animal` and not a `Dog`, it can only access `Animal` methods. Let's add another method to our Dog.

~~~
public class Dog implements Animal{
    public String growl(){
        return 'grrrr';
    }
    public String speak(){
        return 'Bark!'
    }
}
~~~ 

If we try to use the growl method in `AnimalChorus` we'll get an error.

~~~
public class AnimalChorus{
    public static void sing(Animal a){
        system.debug(a.growl());
        //This would not compile, throwing the error:
        //Method does not exist or incorrect signature: [Animal].growl()
    }
}
~~~

So you can think of an interface as a contract; an object is free to have its own unique logic as long as it implements the methods of the interface. This grants us flexibility when we write code. Let's add some other animals to our chorus.


~~~
public class Bird implements Animal{
    public String speak(){
        return 'Tweet!'
    }
}

public class Cat implements Animal{
    public String speak(){
        return 'Meow!'
    }
}

public class Cow implements Animal{
    public String speak(){
        return 'Moo!'
    }
}

//in Execute Anonymous
Dog poochy = new Dog();
Bird tweety = new Bird();
Cat kitty = new Cat();
Cow moomoo = new Cow();

AnimalChorus.sing(poochy);
AnimalChorus.sing(tweety);
AnimalChorus.sing(kitty);
AnimalChorus.sing(moomoo);

//Debug output:
Bark!
Tweet!
Meow!
Moo!
~~~ 

By using an interface, the method `sing` is decoupled from its dependency on the animals. We could change how all the animals sing or add new animals, thus giving `sing` some flexibility in what it does without having to change it at all.

However, with all this flexibility, interfaces themselves are pretty inflexible. If we wanted to add another function to our interface, we have to update EVERY implementation of that interface with that function. It seems like a huge drawback, but in reality you probably won't find it to be much of an issue. The biggest benefit I've found from interfaces comes with testing, which we'll discuss in the next post!