---
layout: post
title: "Getting start with Angular Universal, and avoiding the pitfalls"
image: ""
published: false
---

Angular Universal is a useful tool which allows you to create an Angular app which renders both in the browser and on the server and, unlike a regular Angular app, is able to reap the full benefits of SEO. 

It is reasonably simple to adapt a regular browser rendered Angular app into an Angular Universal app. However, there are some pitfalls which you must avoid, and some small adaptions you must make to the way you write your application code.

In this article I will talk about when to use Angular Universal, how it works and the adaptions you need to make to your codebase, as well as common pitfalls.

## When to use Universal

There are [3 main cited benefits](https://angular.io/guide/universal#why-use-server-side-rendering) of Angular Universal:

1. **To facilitate crawlers from Search Engines and social media sites to crawl your application/site.**

    In reality, this is by far the biggest reason to use Angular Universal. It will allow your application to be indexed by search engines, and allow social media sites such as Facebook to provide a preview of a page of your application when someone posts a link to it. A normal client rendered Angular application will either not be indexed by search engines or have very poor SEO.

    At the time of writing (01/06/19), some crawlers such as the [Google crawler can execute JavaScript](https://seopressor.com/blog/javascript-seo-how-does-google-crawl-javascript/). However, it appears that in practice, people have had mixed results in actually getting their client rendered applications crawled and indexed by Google. Therefore, if you are building an Angular app and SEO is a big concern for you, Angular Universal is your best option.

2. **Improve performance of devices which don't support JavaScript**

    These devices will be unable to run a normal Angular app. With Angular Universal, you provide a server rendered version of the page, without any JavaScript. Whether this no JavaScript version of the page is actually any use to the user will depend on the application in question.

3. **Decrease the (perceived) first page load time**

    With a normal Angular site, when a user first navigates to the site they will have to wait for several hundred KB of JavaScript to download, and then wait for Angular to bootstrap, before they ever see your application.

    When a user first navigates to an Angular Universal appl, the browser first receives a server rendered version of the page that the user navigated to. The user then momentarily sees this JavaScript free, non-interactive, version of the app, while the JavaScript for the Angular app downloads and bootstraps.

## How it works

// TODO

## Getting set up

## Pitfalls and adaptions

## Conclusions