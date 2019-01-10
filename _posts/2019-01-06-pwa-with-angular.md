---
layout: post
title: "PWAs with Angular - Part 2: Building a PWA"
published: false
---

In my previous article, PWAs with Angular - Part 1: Understanding Service Worker, I walked through how to build a simple offline application using plain JavaScript and Service Worker. I recommend reading this as a prerequisite for learning to build PWAs.

Today I will talk about the key features of Progressive Web Apps, as well as how to build a Progressive Web App using Angular, and the tools that we can use to help acheive this. We will build a very simple Angular app, and then do the work necessary to transform it into a PWA.

## What is a PWA

PWAs are applications which run in the browser, but also give the user the option to add the application to their home screen on mobile and tablet - much like a traditional mobile app. They also provide a partially or fully offline experience by caching files using Service Worker and serving up these files when the user is offline.

The key features of a PWA can be summed up as follow:

* **Installable** - when the user visits the app in the browser on their phone or tablet, they get the option to add it to their home screen where it then remains - much like a native app installed from the app store.

* **Offline** - PWAs use Service Worker to provide some level of offline functionality. This may range from a custom offline screen when the network is unavailable, to a full offline experience involving more in depth techniques such as caching data from an API in indexedDb (which I will talk about at the end of this article).

* **Push notifications** - PWAs can send push notifications to the user (this is currently possible on desktop and on Android devices, but not on iOS).

* **Indexable** - unlike native apps, PWAs do not appear in an app store and are simply indexable/searchable, much like websites.

* **Responsive** - As PWAs are intended to work as mobile, tablet and desktop applications, they must work well across a large range of screen sizes.

* **Safe** - PWAs are always served over HTTPS which ensures that the content has not been tampered with.

## When to choose a PWA

Some use cases are better suited to PWAs than others. You may want to choose a PWA based solution when:

* **You want to target Android, iOS and the browser** - while many businesses develop a web application, an Android application and an iOS application for their product, this means maintaining 3 different codebases written in different programming languages. By choosing instead to develop a PWA, we avoid this problem and develop and maintain only a single codebase for an app which works in the browser, on Android, and on iOS.

However, you may not want to choose a PWA as a solution when:

* **You want to support push notifications on iOS** - unlike native iOS apps, PWAs cannot currently send push notifications in iOS.

* **You want your app to appear in Android and iOS app stores** - although PWAs are indexable so appear in search engine results, they cannot be added to the Android or iOS app stores.

## Building the app

First we will build a simple Angular app.

```
ng new pwa-tutorial --style=scss
```

Steps: 

add HttpClient to app.module
set up data service to fetch jsonplaceholeder data
add sone css

PWA:

```
ng add @angular/pwa
```


## Auditing the app, and transforming it into a PWA

## Further steps - fully offline experience with indexedDb