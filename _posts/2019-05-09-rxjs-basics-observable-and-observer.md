---
layout: post
title: "RxJS: demystifying Observables and Observers"
image: ""
---

Observable, observer, subject, producer, subscriber, finite, infinite, unicast, multicast, hot, cold.

There is a lot of terminology which comes with RxJS, and it can feel quite overwhelming. These terms all began to make sense for me when I actually started to take a closer looker at the most basic constructs of this library - the observable and the observer, really work.

In this article

# Observer pattern

The [Observer Pattern](https://sourcemaking.com/design_patterns/observer) is one of the most well known patterns in software development. A nice definition of the Observer pattern from O'Reillys Head First Design Patterns is as follows:

The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated immediately.

In this pattern the following terminology is used to describe the actors involved:

* Subject - this is an object which stores or accesses data and provides methods via which interested parties can subscribe and be notified of changes to this data. This succession of notifications can also be thought of as a stream of events.
* Observers - these are objects which choose to subscribe to a Subject and recieve notifications when its data changes. 

So how does this relate RxJS?

The relationship between RxJS Observables and Observers conforms exactly to the pattern described above, with the Observable being the Subject. Where this gets slightly confusing is that there is also an RxJS Subject (itself a Subject in the sense of the Observer pattern), which I will touch on later in this article.

# RxJS Observables and Observers

## Observable

An **observable** is best conceptualised as a stream of events, to which its **observers** can subscribe.

The observable can essentially do three things in terms of notifying its observers.

1. Emit a value - (0, 1 or many times)
2. Emit an error - (0 or 1 times)
3. Complete - (0 or 1 times)

Once an observable either errors or completes, it will unsubscribe all of its Observers and can no longer emit values, error or complete. A consequence of this is that an observable will never both error and complete.

## Observer

An **observer** is simply an object with `next`, `error` and `complete` methods which handle notifications from an observable.

```ts
const observer = {
    next: (value) => {},
    error: (error) => {},
    complete: () => {}
}
```

## In practice

So how do we create and work with RxJS Observables? Lets have a look.

```ts
import { Observable } from 'rxjs';

/*
 * A 'subscription' function, defining what action to take each time 
 * an observer subscribes. The function will be passed the 
 * subscribing observer as a parameter.
 */
const subscriptionFn = observer => {
  console.log('A new subscriber');
  observer.next(1);
  observer.next(2);
  observer.complete();
}

/*
 * Create a new Observable, giving it a subscription function to 
 * execute whenever any Observer subscribes.
 */
const source$ = Observable.create(subscriptionFn);

/*
 * An Observer - simply an Object with next, error and complete
 * methods.
 */
const observer = {
    next: value => console.log(value),
    error: error => console.log(`Error: ${error}`),
    complete: () => console.log('Complete'),
};

/*
 * We subscribe here, causing our observable to execute the 
 * subscription function we gave it earlier, with the observer 
 * we give it here as an argument.
 */
source$.subscribe(observer);
// A new subscriber
// 1
// 2

// We subscribe again, causing the subscription function to execute again.
source$.subscribe(observer);
// A new subscriber
// 1
// 2
```

We use 2 important methods here:

1. **Observable.create**

    RxJS provides an `Observable.create` method which is used, unsurprisingly, to create an Observable (there are other ways to create Observables, which I will talk about further on). 
    
    `Observable.create` method takes a single parameter, `subscribe`, which is a function that defines what action to take each time an observer subscribes. This function is referred to as the observable's **subscription function** (sometimes it is also known as the producer function).
    
    Essentially, when we call `Observable.create`, we are saying 'create me a new Observable, and execute this subscription function each time any Observer subscribes to the Observable'.

    A consequence of this is that the subscription function is not executed when we call `Observer.create`, only upon subscription - this behaviour is known as lazy execution.

2. **Observable.subscribe**

    `subscribe` is the method via which an observer can register with an observable in order to listen to its notifications. 
    
    `subscribe` takes a single argument - an observer, as well as a number of shorthands which allow the consuming code to pass in only the methods of the observer that it wishes to implement.

    ```ts
    source$.subscribe(v => console.log(v));
    ```

    This is equivalent to:

    ```ts
    const observer = {
        next: v => console.log(v),
    }

    source$.subscribe(observer);
    ```

    Which is in turn equivalent to:

    ```ts
    const observer = {
        next: v => console.log(v),
        error: () => {},
        complete: () => {},
    }

    source$.subscribe(observer);
    ```

# Finite and Infinite Observables

## Finite Observable

A **finite observable** is an observable which completes.

Think of an observable which wraps a http request. This Observable will always emit a single value and then complete (or it will error).

The timeline for this observable will look like the below:

**-------------------xc**

Or in the case of an error:

**---------------------e**

There is no need to unsubscribe from finite observables, this is because when an observable completes or errors, it will automatically unsubscribe any subscribers on its own.

## Infinite Observable

An **infinite Observable** is an Observable which never completes.

Think of an observable which wraps a sequence of mouse click events. There is no termination to this sequence, and hence the observable will not complete.

The timeline for this Observable will look like the below:

**--x---x---x----x---**

The fact that the observable never completes means that any observers will hang around in memory for the lifespan of the observable. This is why it is often emphasized that you should unsubscribe from infinite observables when once the observer no longer needs to receive its emissions.


## Hot and Cold Observables

First, lets define a **producer** as a source of values for an observable - ie. the thing which actually calls the observer's `next`, `complete` and `error` methods.

As we will see now, the producer can either be the producer function itself, or some source contained within or closed over by the producer function (eg. a sequence of DOM events, a http response, a connection to a web socket).

### Cold Observable

A cold observable has a producer function which creates a producer each time it executes.

To understand what this looks like in practice, lets look at an example of a cold observable which wraps a http request made with the fetch api:

```ts
const producer = observer => {
  const myPromise = fetch('someUrl'); 
  myPromise.then((data) => {
    observer.next(data);
    observer.complete();
  }, (err) => {
    observer.error(err);
  });
}

const source$ = Observable.create(producer);
```

Lets break down what happens here into a number of steps:

1. We define our producer function.
2. We create an observable using the producer function that we defined.
3. We subscribe to the observable. At this point, our producer function executes. We make a http request via the fetch api (which is the producer in this case) and then pass the response to the observer.
4. We create another subscription to the same observable. Our producer function executes again, and we make another http request via the fetch api and pass this second response to our second observer.

In this case note that:

* We make 1 http request per subscriber. 
* Values emitted by the observable are not shared between observers. An Observable which behaves this way is known as **unicast**.

### Hot Observable

A hot observable has a producer function which closes over its producer.

To understand how this is different to the cold observable, lets look at an example of a hot observable which wraps a http request made with the fetch api:

```ts
// Promise created outside of producer function, the producer function closes over the promise.
const myPromise = fetch('someUrl'); 

const producer = observer => {
  myPromise.then((data) => {
    observer.next(data);
    observer.complete();
  }, (err) => {
    observer.error(err);
  });
}

const source$ = Observable.create(producer);
```

Lets do the same thing as above, and break this down into a number of steps.

1. We make a http request via the fetch api and store this as a promise.
1. We define our producer function.
2. We create an Observable using the producer function that we defined.
3. We subscribe to the Observable. at this point, our producer function executes. We pass the resolved value of our promise to the Observer
4. We create another subscription to the same Observable. Our producer function executes again, and we pass the same resolved value to our second Observer.

In contrast to the cold observable:

* We only make 1 http request. 
* Values emitted by the observable are shared between observers. An observable which behaves this way is known as **multicast**.

# Conclusion

* https://medium.com/@benlesh/learning-observable-by-building-observable-d5da57405d87
* https://medium.com/@benlesh/hot-vs-cold-observables-f8094ed53339
* https://netbasal.com/whos-afraid-of-observables-bde0dc4f48cc