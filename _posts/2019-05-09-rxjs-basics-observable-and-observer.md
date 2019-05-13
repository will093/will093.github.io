---
layout: post
title: "RxJS basics: Observables, Observers and Operators"
image: ""
---

Observable, Observer, Subject, Operator, Producer, Subscriber. 

When I first started learning RxJS, it took me some time to figure out exactly what exactly all of these things were. There was a lot of new terminology, and I felt quite overwhelmed. Almost 2 years down the line, although there are plenty of operators I have yet to explore, at a conceptual level I have a much better grasp on RxJS.

In this article I hop to sum up some of the key terminology for developers who are just getting started with RxJS.

# Observer pattern

The [Observer Pattern](https://sourcemaking.com/design_patterns/observer) is one of the well known patterns in software development. A nice definition of the Observer pattern from O'Reillys Head First Design Patterns is as follows:

The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated immediately.

In this pattern the following terminology is used to describe the actors involved:

* Subject - this is an object which stores or accesses data and provides methods via which interested parties can subscribe and be notified of changes to this data. This succession of notifications can also be thought of as a stream of events.
* Observers - these are objects which choose to subscribe to a Subject and recieve notifications when its data changes. 

So how does this relate RxJS?

The relationship between RxJS Observables and Observers conforms exactly to the pattern described above, with the Observable being the Subject. Where this gets slightly confusing is that there is also an RxJS Subject (itself a Subject in the sense of the Observer pattern), which I will touch on later in this article.

# RxJS Observables and Observers

## Observable

An Observable is best conceptualised as a stream of events, to which its Observers can subscribe.

The Observable can essentially do three things in terms of notifying its Observers.

1. Emit a value - (0, 1 or many times)
2. Emit an error - (0 or 1 times)
3. Complete - (0 or 1 times)

Once an Observable either errors or completes, it will unsubscribe all of its Observers and can no longer emit values, error or complete. A consequence of this is that an Observable will never both error and complete.

## Observer

An Observer is simply an object with `next`, `error` and `complete` methods which handle notifications from an Observable.

```ts
const observer = {
    next: (value) => {},
    error: (error) => {},
    complete: () => {}
}
```

## In practice

So how do we create and work with RxJS Observables? Lets first have a look at a snippet of code.

```ts
import { Observable } from 'rxjs';

const source$ = Observable.create(observer => {
  observer.next(1);
  observer.next(2);
  observer.complete();
});

const observer = {
    next: value => console.log(value),
    error: error => console.log(`Error: ${error}`),
    complete: () => console.log('Complete'),
};

source$.subscribe(value => {
  console.log(value) // 1, 2
});
```

We use 2 important methods here:

1. **Observable.create**

    RxJS provides an `Observable.create` method which is used, unsurprisingly, to create an Observable (there are other ways to create Observables, which I will talk about further on). 
    
    `Observable.create` method takes a single parameter, `subscribe`, which is a function that allows us to emit data to Observers by directly making calls to an Observer's methods. This function is sometimes referred to as the Observable's **producer function**.

2. **Observable.subscribe**

    Subscribe is the method via which Observers can register with an Observable in order to listen to its notifications.


## Producer

## Finite and infinite Observables

A finite Observable is one which either error or completes. 

## Observer



## Operator

## Subscription

// TODO StackBlitz example, imports etc.
// TODO RxJS v5/v6