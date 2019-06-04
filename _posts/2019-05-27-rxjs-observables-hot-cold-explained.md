---
layout: post
title: "RxJS: Hot, Cold, Finite, Infinite, Unicast and Multicast Observables explained"
image: "/images/rxjs-observables-hot-cold-explained/marbles.jpg"
published: true
---

Observable, observer, producer, subscriber, finite/infinite, unicast/multicast, hot/cold, subject, operator, ... and so on.

There are a lot of new concepts and terminology that come with RxJS, and it can be tricky to get your head around all of it. 

In my opinion, the best way to fully understand these concepts is to go back to basics and take a more detailed look at how observables and observers really work. From this, many of the more advanced RxJS concepts follow quite naturally.

In this article, we will first look at the Observer pattern, which RxJS Observables/Observers are an implementation of. Then, we will take a deeper look at how observables and observers work, and from this, arrive at an understanding of what it means when we say an observable is finite, infinite, hot, cold, unicast, or multicast.

# Observer pattern

The [Observer pattern](https://sourcemaking.com/design_patterns/observer) is one of the most well known patterns in software development. A nice definition of the observer pattern from O'Reilly's *Head First Design Patterns* is as follows:

> The observer pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated immediately.

In this pattern the following terminology is used to describe the actors involved:

* ***Subject*** - this is an object which stores or accesses data and provides methods via which interested parties can subscribe and be notified of changes to this data. This succession of notifications can also be thought of as a stream of events.
* ***Observers*** - these are objects which subscribe to a *subject* and recieve notifications when its data changes. 

So how does this relate to RxJS?

The relationship between RxJS Observables and Observers conforms exactly to the pattern described above, with the RxJS Observable being the *subject* and the RxJS Observers being the *observers*. 

Where this gets slightly confusing is that there is also an RxJS Subject. Here we use the term *subject* to refer only to its definition as given above, in the context of the Observer pattern. 

# RxJS Observables and Observers

## Observable

An **observable** is best conceptualised as a stream of events, to which its **observers** can subscribe.

The observable can essentially do three things in terms of notifying its observers.

1. Emit a value - (0, 1 or many times)
2. Emit an error - (0 or 1 times)
3. Complete - (0 or 1 times)

Once an observable either errors or completes, it will unsubscribe all of its observers and can no longer emit values, error or complete. A consequence of this is that an observable will never both error and complete.

## Observer

An **observer** is simply an object with `next`, `error` and `complete` methods which handle notifications from an observable.

```ts
const observer = {
    next: (value) => {},
    error: (error) => {},
    complete: () => {}
}
```

## Observables in practice

So how do we create and work with observables? Lets have a look at a simple example.

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
 * Create a new observable, giving it a subscription function to 
 * execute whenever any Observer subscribes.
 */
const source$ = Observable.create(subscriptionFn);

/*
 * An observer - simply an object with next, error and complete
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

We use two important methods here:

1. **Observable.create**

    RxJS provides an `Observable.create` method which is used, unsurprisingly, to create an observable. 
    
    The `Observable.create` method takes a single parameter, `subscribe`, which is a function that defines what action to take each time an observer subscribes. This function is referred to as the observable's **subscription function** (sometimes it is also known as the producer function).
    
    Essentially, when we call `Observable.create`, we are saying 'create me a new observable, and execute this subscription function each time any observer subscribes to the observable'.

    A consequence of this is that the subscription function is not executed when we call `Observer.create`, but instead executes when an observer subscribes to the observable - this behaviour is known as **lazy execution**.

2. **Observable.subscribe**

    `subscribe` is the method via which an observer can register with an observable in order to listen to its notifications. This method takes a single argument - an observer, as well as a number of shorthands which allow the consuming code to pass in only the methods of the observer that it wishes to implement.

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

Think of an observable which wraps a http request. This observable will always emit a single value and then complete (or it will error). 

We can visualise this observable stream as follows:

**- - - - - - - - - - xc**

Or in the case of an error:

**- - - - - - - - - - - e**

There is no need to explicitely unsubscribe from finite observables, this is because when an observable completes or errors, it will automatically unsubscribe any subscribers on its own.

## Infinite Observable

An **infinite Observable** is an observable which never completes.

Think of an observable which wraps a sequence of mouse click events. There is no termination to this sequence, and hence the observable will not complete.

We can visualise this observable stream as:

**- x - - x - - x - - x - -**

The fact that the observable never completes means that any observers will hang around in memory for the lifespan of the observable. This is why it is often emphasized that you should unsubscribe from infinite observables when once the observer no longer needs to receive its emissions.

In practice, the consumer doesn't always know whether the observable it is subscribing to is finite or infinite, and in these cases it is best to explicitely unsubscribe.

# Hot and Cold Observables

First, lets take a look at **producers**.

A **producer** as a source of values for an observable - ie. the thing which actually calls the observer's `next`, `complete` and `error` methods.

The producer can either be the subscription function itself, or some source contained within or closed over by the subscription function (eg. a sequence of DOM events, a http response, a connection to a web socket).

## Cold Observable

A **cold observable** has a subscription function which creates a producer each time it executes.

To understand what this looks like in practice, lets look at an example of a cold observable which wraps a Promise returned from the fetch api:

```ts
const subscriptionFn = observer => {
  const responsePromise = fetch('someUrl'); 
  responsePromise.then((data) => {
    observer.next(data);
    observer.complete();
  }, (err) => {
    observer.error(err);
  });
}

const source$ = Observable.create(producer);

source$.subscribe(response => console.log(response));
source$.subscribe(response => console.log(response));

```

Lets break down what happens when this executes into a series of chronological steps:

1. We define our subscription function.
2. We create an observable using the subscription function that we defined.
3. We subscribe to the observable. At this point, our subscription function executes. We make a http request via the fetch api (the producer) and then pass the http response to the observer.
4. We subscribe another observer to the same observable. Our subscription function executes again, and we make another http request via the fetch api and pass this second response to our second observer.

In this case note that:

* We make 1 call to the fetch api and hence 1 http request per observer, when that observer subscribes (lazy execution). 
* Values emitted by the observable are not shared between observers (ie. each observer receives a different `response` object). An Observable which behaves in this way is known as **unicast**.

## Hot Observable

A hot observable has a subscription function which closes over its producer.

To understand how this is different to the cold observable, lets look at an example of a hot observable which wraps a Promise returned from the fetch api:

```ts
/*
 * The http request is made outside of the subscription function, 
 * and so it executes eagerly rather than lazily.
*/
const responsePromise = fetch('someUrl'); 

const subscriptionFn = observer => {
  responsePromise.then((data) => {
    observer.next(data); * 
    observer.complete();
  }, (err) => {
    observer.error(err);
  });
}

const source$ = Observable.create(subscriptionFn);

source$.subscribe(response => console.log(response));
source$.subscribe(response => console.log(response));
```

Lets do the same thing that we did for the cold observable, and break down what happens when this executes into a series of chronological steps:

1. We make a http request via the fetch api and store this as a promise.
2. We define our subscription function.
3. We create an observable using the subscription function that we defined.
4. We subscribe to the observable. At this point, our subscription function executes. We instruct our promise to pass the resolved value of our promise to the observer once it resolves.
5. We subscribe another observer to the same observable. Our subscription function executes again, and we again instruct the same promise to pass its resolved value to our second observer.

In contrast to the cold observable:

* We only make 1 call to the fetch api and hence 1 http request, before the observable is ever created (eager execution). 
* Values emitted by the observable are shared between observers (ie. each observer receives the same `response` object). An observable which behaves this way is known as **multicast**.

# Conclusions (tl;dr)

We have taken a look at how RxJS Observables and Observers relate to the Observer pattern, had a deeper looker at how observables really work, and also at some of the terminology surrounding them. 

To finish up, here is a quick one line summary of each of the terms that we have looked at:

* **Observable** - a stream of events to which observers can subscribe.
* **Observer** - an object with `next`, `error` and `complete` methods, which subscribes to an observable.
* **Subscription function** - the function which executes each time an observer subscribes to an observable.
* **Producer** - the source of data for an observable, the thing that calls an observers `next`, `error`, and `complete` methods. 
* **Hot observable** - an observable which creates its producer.
* **Cold observable** - an observable which closes over its producer.
* **Finite observable** - an observable which completes.
* **Infinite observable** - an observable which never completes.
* **Unicast observable** - an observable whose emitted values **are not** shared amongst subscribers.
* **Multicast observable** - an observable whose emitted values **are** shared amongst subscribers.

# Helpful resources

The following were really helpful for writing this article, and I very much recommend them as further reading:

* <https://medium.com/@benlesh/learning-observable-by-building-observable-d5da57405d87>
* <https://medium.com/@benlesh/hot-vs-cold-observables-f8094ed53339>
* <https://netbasal.com/whos-afraid-of-observables-bde0dc4f48cc>
