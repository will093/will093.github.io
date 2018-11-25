---
layout: post
title: Understanding and implementing a Service Worker
published: false
---

A Service Worker is a type of Web Worker, which runs in a thread separate to the main browser thread. It sits between an application, the browser, and the network.

The most common use of Service Worker is for caching files and assets in partially or fully offline web applications. In this article, I will explain the key necessary steps for doing this.

The demo application from which all the code snippets in this article are taken from can be found here: https://github.com/will093/service-worker-basic

## Service Worker lifecycle events
Service Worker has 2 important lifecycle events which we must implement when creating offline applications.

### Install

Installation occurs when we first register a Service Worker, and then again every time the Service Worker file is updated, (a Service Worker is considered updated if it is not byte equal to the currently installed Service Worker).

During installation we cache any static files such as HTML, CSS, JS and images, so that on subsequent loads they are available offline. Service worker uses the Cache API to store these files. Note that you must place your service worker in or above the directory of the files that you wish to cache.

```javascript
// Install is where we cache the static assets belonging to the application.
self.addEventListener('install', function (event) {
  console.log('Installing SW');
  // Wait until SW has finished caching in order for it to be considered installed.
  event.waitUntil(
      // Open the cache and add our static files.
      caches.open(currentCacheNames)
          .then((cache) => {
              return cache.addAll([
                './',
                './index.html',
                './scripts/main.js',
                './styles/main.css',
                '/bower_components/jquery/dist/jquery.js',
                '/bower_components/modernizr/modernizr.js'
              ]);
          })
  );
});
```

After installation our Service Worker must still wait to be activated before network requests will be forwarded to it.

### Activate

On activation, we clean up the cached files belonging to any previous version of the service worker.

* If we are loading the page for the first time, (and so there is no existing service worker) then activation will take place after installation is finished.

* If there is already an active Service Worker on the page, activation of the new service worker will only happen when there is no other instance of the app is running in any other tab or window.

```javascript
After installation our Service Worker must still wait to be activated before network requests will be forwarded to it.

Activate
On activation, we clean up the cached files belonging to any previous version of the service worker.

If we are loading the page for the first time, (and so there is no existing service worker) then activation will take place after installation is finished.
If there is already an active Service Worker on the page, activation of the new service worker will only happen when there is no other instance of the app is running in any other tab or window.
```

There are a few further caveats with activation, primarily:

1) Refreshing the page will not be enough to activate the Service Worker even if the app is not open in any other tab — when you refresh the page there is some overlap during which both the new and old page are open. To activate the Service Worker you can navigate to another page and then back again.

2) The first time you load a particular page which has a service worker, the service worker will still not intercept network requests even after activation, this is because if a page was not loaded with a service worker then neither are its subresources.

## The fetch event

Once our Service Worker is activated, this event is triggered any time an asynchronous XHR request or Fetch API request is made, and is where we write logic for determining whether to fetch the response from the cache or forward the request to the network/server.

As an example, here is a simple implementation which checks the cache and returns the cached response if it exists, or forwards the request to the network otherwise:

```javascript
self.addEventListener('fetch', function (event) {
  event.respondWith(
    // Return the file from the cache if it exists, 
    // otherwise forward the request to the network.
    caches.match(event.request).then((response) => {
        return response || fetch(event.request);
    })
  )
});
```

Note that the above is a very trivial example and is only suitable for the circumstance where none of the cached files ever get updated.

It is really up to you as a developer to decide on the caching strategy that you wish to use when a request is made — https://serviceworke.rs has many good examples of caching strategies.

