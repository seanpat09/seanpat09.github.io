---
layout: post
title: LWC Testing - Mocking registerListener in pubsub
date: '2019-06-05-T00:00:00.000-07:00'
author: Sean Cuevo
tags:
- salesforce
- lwc
- testing
---

Communication to sibling components in LWC is not included out of the box, but luckily the [lwc-recipes GitHub repo](https://github.com/trailheadapps/lwc-recipes/tree/master/force-app/main/default/lwc/pubsub) contains a pubsub component that can handle this for you. Testing these custom events, however, was a little tricky and I wanted to share how I approached it.

Let's say you have two components - one that displays data and one that displays a status. When the data is retrieved from the Apex controller, you want to fire an event so that the status component knows to display the status. With pubsub, you can fire an event from the data component and have the status component listen for that event. Now your two components can communicate while also staying loosely coupled. Here's a simple version of what the JavaScript would look like for each component.


```
//c-data-component

import { LightningElement, wire } from 'lwc'
import { CurrentPageReference } from 'lightning/navigation'
import getData from '@salesforce/apex/DataController.getData'

import { fireEvent } from 'c/pubsub'

export default class DataComponent extends LightningElement {
    @wire(CurrentPageReference) pageRef;

    fetchData() {
        getData()
            .then(
                function() {
                    fireEvent(this.pageRef, 'myevent')
                }.bind(this)
            );
    }
}
```

```
//c-status-component

import { LightningElement, wire, track } from 'lwc'
import { CurrentPageReference } from 'lightning/navigation'

import { fireEvent, registerListener, unregisterAllListeners } from 'c/pubsub'

export default class DataComponent extends LightningElement {
    @wire(CurrentPageReference) pageRef;

    connectedCallback() {
        registerListener('myevent', this.displayStatus, this);
    }

    disconnectedCallBack() {
        unregisterAllListners(this);
    }
    
    displayStatus() {
        this.displayStatus = true;
        this.template.querySelector('.status').classList.remove('slds-hide');
    }
}

//html

<template>
    <div class="status slds-hide">Data Retrieved!></div>
<template>
```

In summary:

* When `fetchData()` is called by `c-data-component`, it fires the `myevent` event
* When `c-status-component` is initialized, it registers a listener for `myevent`
* When `myevent` is fired, `c-status-component` should call `displayStatus` and display the status div

As a part of the `c-status-component` lwc-jest tests, we should confirm that the listener is registered, and that when the `myevent` is triggered, the status div is displayed. Testing that the listeners are registered is fairly straightforward, there are a lot of examples of this in lwc-recipes.

```
import { createElement } from 'lwc';
import StatusComponent from 'c/statusComponent'
import { registerListener, unregisterAllListeners } from 'c/pubsub'

//mock the pubSub methods
jest.mock('c/pubsub', () => {
    return {
        registerListener: jest.fn(),
        unregisterAllListeners: jest.fn()
    }
})

//remove all elements from test DOM after each test
afterEach(() => {
    while (document.body.firstChild) {
        document.body.removeChild(document.body.firstChild);
    }
    jest.clearAllMocks();
})

describe('Listeners', () => {
    it('should register and unregister myevent listener', () => {
        let element = createElement('c-status-component', { is: StatusComponent }
        document.body.appendChild(element);

        expect(registerListener.mock.calls.length).toBe(1);
        expect(registerListener.mock.calls[0][0]).toEqual('myevent');

        document.body.removeChild(element);
        expect(unregisterAllListeners.mock.calls.length).toBe(1)
    })
})
```

This is pretty boiler plate for any component that you want to use pubsub with. However, I was a little puzzled on how also wanted to test displaying the status. If registerListener is being mocked, then even if I could figure out how to fire this custom event, the mock wouldn't fire the callback. Luckily, Jest mocks allow you to access the parameters used to call a mocked method.

So I figured a good test would be to intercept the callback parameter, fire it manually in the test, which would adequately simulate the pubsub listener firing the callback.

```
describe('Listeners', () => {
    ...
    it('should display the status when the myevent is fired', () => {
        //Access the first call of registerListener and get the 2nd parameter, which is the callback
        let myeventCallback = registerListener.mock.calls[0][1];

        //Access the first call of registerListener and get the 3rd parameter
        //which is what `this` should be bound to
        let thisArg = registerListener.mock.calls[0][1];

        let element = createElement('c-status-component', { is: StatusComponent }
        document.body.appendChild(element);

        //fire the callback
        myeventCallback.call(this)

        //return a promise to resolve DOM changes
        return Promise.resolve().then() => {
            const statusDiv = element.shadowRoot.querySelector('.status');
            expect(statusDiv.classList).not.toContain('slds-hide');
        }
    })
})
```

When `document.body.appendChild(element);` is called, that will fire `connectedCallback`, and the mock for registerListener will intercept the parameter `this.displayStatus`. Now you can access that parameter and call the method with an explicitly set `this` argument. Hopefully this helps you when writing tests on pubsub events!

