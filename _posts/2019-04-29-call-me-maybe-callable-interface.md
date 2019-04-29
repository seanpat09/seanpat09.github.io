---
layout: post
title: Call Me Maybe - Using the Callable Interface to Build Versioned APIs
date: '2019-04-29-T00:00:00.000-07:00'
author: Sean Cuevo
tags:
- salesforce
- apex
- interfaces
- callable
---

In Winter '19, Salesforce introduced the [Callable Interface.](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_interface_System_Callable.htm)

>Enables developers to use a common interface to build loosely coupled integrations between Apex classes or triggers, even for code in separate packages. Agreeing upon a common interface enables developers from different companies or different departments to build upon one another’s solutions. Implement this interface to enable the broader community, which might have different solutions than the ones you had in mind, to extend your code’s functionality.

In short, you implement the interface and its single `call` method, pass it the name of the action you want to call and a `Map<String,Object>` with any necessary parameters, which dispatches the logic from there.

```
public class CallMeMaybe implements Callable {
    public Object call(String action, Map<String, Object> args) {
        switch on action {
            when 'doThisThing' {
                service.doThisThing();
            }

            when 'doThatThing' {
                service.doThatThing();
            }
        }
        return null;
    }
}
```

```
public class Caller {
    public void callTheCallable() {
        if (Type.forName('namespaced__CallMeMaybe') != null) {
            Callable extension = (Callable) Type.forName('namespaced__CallMeMaybe').newInstance();
            extension.call('doThisThing', new Map<String,Object>());
        }
    }
}
```

There's nothing too novel here other than the conveniences this new standard interface gives us, the largest being the ability to execute methods in other packages without having a hard dependency on that package. What jumped out to me, however, was the idea of dispatching actions using a string parameter and how we can use that to build more flexible APIs in managed packages.

## Versioned APIs

One way to expose a method for execution in a managed package is to mark it as global. These global methods serve as an API to your package. However, if you ever wanted to adjust the behavior of a global method, you risked causing unintended side affects on subscribers that depend on the original implementation. To get around this, I generally see packages create additional global methods with names like `myMethodV2`.

The finality of global methods tend me to make me agonize over creating them. Yes, you can deprecate them, but it felt like you were polluting your namespace. `myMethodV2` may seem ok, but `myMethodV16` starts to feel a little messy. Did you know there are 15 *The Land Before Time* movies? It's not a good look.

Instead, what if you created a single Callable entry point into your org as an API?

```
public class VersionedAPI implements Callable {
    public Object call(String action, Map<String, Object> args) {
        //format actions using the template "domain/version/action"
        //e.g. "courses/v1/create"

        List<String> actionComponents = action.split('/');
        String domain = actionComponents[0];
        String version = actionComponents[1];
        String method = actionComponents[2];

        switch on domain {
            when 'courses' {
                return courseDomain(version, method, args);
            }

            when 'students' {
                return studentDomain(version, method, args);
            }

            ...
        }
        return null;
    }

    public Object courseDomain(String version, String method, Map<String, Object> args) {
        if (version == 'v1') {
            switch on method {
                when 'create' {
                    return courseServiceV1.create();
                }
                ...
            }
        } else if (version == 'v2') {
            switch on method {
                when 'create' {
                    return courseServiceV2.create();
                }
                ...
            }
        }
    }

    ...
}
```

By following this pattern, you'll have a little more flexibility in defining your exposed methods without having to worry about the permanence of that method. 

* Typos in your action names aren't forever anymore!
* Remove actions that you don't need. No more ghost town classes filled with `@deprecated` methods
* Use new versions to change an actions behavior while allowing your subscribers to update their references at their convenience
* Experiment with new API actions in a packaged context without fear of them living in the package forever if you change your mind

Of course, with this added flexibility comes the burden of communicating these changes out to your subscribers - if you remove an action, make sure to have a migration plan in place so your subscribers aren't suddenly faced with a bug that you introduced. By following this pattern, however, I hope it will encourage more developers to expose more functionality as well as foster inter-package testing.