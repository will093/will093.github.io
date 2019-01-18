---
layout: post
title: "PWAs with Angular - Part 1: Understanding Service Worker"
image: "/images/understanding-sw/cogs.jpg"
---

Recently, I have been hearing a lot of hype around Progressive Web Apps (PWAs). At first glance PWAs seem rather amazing and revolutionary - something that can be deployed as and function as a web application, but which also works offline, and can be added to the home screen like a native app on both Android and iOS.

One of the key characteristics of a PWA is its offline functionality, which is implemented with Service Worker. Understanding Service Worker is probably the most difficult hurdle you must overcome in order to learn to build Progressive Web Apps. 

In this article we will build a small example app, and then add in offline functionality using Service Worker. This will give us a good understanding of the lower level APIs which the Angular PWA utilities are built on top of, and be helpful later on when it comes to debugging PWAs.

## What is a Service Worker

A Service Worker is a type of Web Worker, which runs in a thread separate to the main browser thread. It sits between an application, the browser, and the network.

The most common use of Service Worker is caching files and assets in partially or fully offline web applications.

## Getting started

The demo application from which all the code snippets in this article are taken from can be found here: <https://github.com/will093/service-worker-tutorial>

We will start by creating a basic web application using a Yeoman generator, and then explore how we can add offline functionality using Service Worker.

```
npm install --global yo gulp-cli bower generator-webapp
mkdir service-worker-tutorial
cd service-worker-tutorial
yo webapp
```

You will be presented with a series of options for the project - when prompted, choose none of the additional features, say no to jQuery, and choose either TDD or BDD on the last question.

Now to run our application and automatically open the browser:

```
gulp serve
```

Now if we open the Chrome devtools and look in the network tab, we will see the following:

![Files to cache](/images/understanding-sw/files-to-cache.png)

What is important here are those first 3 files - these are the HTML, CSS and JavaScript which make up our web application. They are the files which we wish serve up from the cache when the user is offline.

## Adding and registering a service worker

Inside of the app folder, add a file named `sw.js`, this is our Service Worker. Then go into `main.js` and replace the contents with the following:

```javascript
// Check to see if your browser supports Service Worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker
      // Register the Service Worker.
      .register('sw.js')
      // Using promises tell us if successful or there was an error
      .then(reg => {console.info('Service Worker registration successful: ', reg)})
      .catch(err => {console.warn('Service Worker setup failed: ', err)});
}
```

In the above we are checking if the user's browser supports Service Worker, and then registering the Service Worker (which is just a JavaScript file) if it does.

If you now refresh the browser and look in the console, you will see that the Service Worker gets succesfully registered. In fact - you will see that every time the page is loaded, the Service Worker appears to get registered again. Actually, while the first time the Service Worker is registered it will trigger the 'install' lifecycle event, on subsequent reloads `serviceWorker.register()` is actually a no-op.

## Implementing the install and activate lifecycle events

Service Worker has 2 important lifecycle events - `install` and `activate` - which we must implement when creating offline applications.

### Install

Installation occurs when we first register a Service Worker, and then again every time the Service Worker file is updated, (a Service Worker is considered updated if it is not byte equal to the currently installed Service Worker).

During installation we cache any static files such as HTML, CSS, JS and images, so that on subsequent loads they are available offline. Service worker uses the Cache API to store these files.

Open up `sw.js` and add the following code:

```javascript
'use strict';

// We use a singular, versioned cache. We give the cache a version, as each subsequent version of the service worker will usually correspond to updates to our set of cached assets.
const currentCacheNames = ['v1'];

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
                './scripts/main.js',
                './styles/main.css',
              ]);
          })
  );
});
```

If we then refresh the browser and look in the console output, we will see that our install event handler has been triggered.

![Service Worker Install](/images/understanding-sw/sw-install.png)

And if we look in the Service Workers section in the Application tab, we can see the current status of any Service Workers on the domain.

![Application Tab](/images/understanding-sw/sw-application.png)

In fact, in the Application tab, we can open up and view Cache Storage - this is where Service Worker caches files. If we open up the cache named 'v1' (which is the cache name we specified earlier) then we can see that our HTML, CSS and JS files are cached here as expected.

![Application Tab Cach Storage](/images/understanding-sw/sw-application-cache-storage.png)

If we refresh the page again, we see that the install event handler doesn't run again - this is expected as the Service Worker has not been updated. 

Now that our Service Worker is installed, we must still wait for it to be activated before network requests will be forwarded to it.

### Activate

On activation, we clean up the cached files belonging to any previous version of the service worker.

* If we are loading the page for the first time, (and so there is no existing service worker) then activation will take place after installation is finished.

* If there is already an active Service Worker on the page, activation of the new service worker will only happen when there is no other instance of the app is running in any other tab or window.

Open up `sw.js` and append the following code at the bottom of the file:

```javascript
// Activate is where we clean up the caches of any previous versions of the SW.
self.addEventListener('activate', function (event) {
  console.log('Activating SW');
  event.waitUntil(
      // Get all of the caches by key.
      caches.keys()
          .then((caches) => {
              return Promise.all(
                  caches.map((cacheName) => {
                      // Delete any caches which don't belong to the current version of the SW.
                      if (currentCacheNames.indexOf(cacheName) === -1) {
                          return caches.delete(cacheName);
                      }
                  })
              );
          })
  );
});

```

Now if we refresh the page once more we will see that the install event handler runs, but the activate event handler does not. This is because we just updated our Service Worker file - meaning that the new version has been installed, but cannot be activated while the app is running in any open tab.

We can confirm that our Service Worker has not yet activated by opening up the Service Workers section in the Application tab in the Chrome devtools.

![Service Worker Application Tab Waiting](/images/understanding-sw/sw-application-waiting.png)

We did refresh the page, so you would think that there was a time when no instance of the app was running - but this is not the case. Refreshing the page will not be enough to activate the Service Worker even if the app is not open in any other tab - when you refresh the page there is some overlap during which both the new and old page are open. 

To activate the Service Worker you can navigate to another page and then back again, and this will trigger the 'activate' event.

![Service Worker Activate](/images/understanding-sw/sw-activate.png)

## The fetch event

Once our Service Worker is activated, the 'fetch' event is triggered any time an asynchronous XHR request or Fetch API request is made, and is where we write logic for determining whether to fetch the response from the cache or forward the request to the network/server.

Again, open up `sw.js` and append the following code at the bottom of the file:

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

Now if we navigate away from the page and then navigate back again, the updated Service Worker will be activated. If we open up the network tab in the dev tools then we will see the following:

![Service Worker Caching](/images/understanding-sw/sw-caching.png)

The Service Worker is intercepting each request and making either making its own request to the network (the requests with the cog icon next to them are made by Service Worker) or just retrieving files from the cache without making a request to the network (the files which say '(from ServiceWorker)' in the size column and have no request made by Service Worker).

## Caching strategies

There are many different caching strategies that you can use. In our example we simply retrieve the file from the cache if it is available, and only ever use the network to retrieve the file if it is not in the cache. This caching strategy is only suitable for the circumstance where none of the cached files ever get updated. 

You will usually want to implement a more complex caching strategy, and this strategy will be dependent on your use case. For example you may wish to attempt to get the freshest version of the files from the network, but then if the network is unavailable or is taking too long, retrieve the files from the cache. You may also wish to use different caching strategies for different file types.

It is really up to you as a developer to decide on the caching strategy that you wish to use — https://serviceworke.rs has many good examples of caching strategies.

## Browser Support

Service Workers are supported in the latest versions of Edge, Firefox, Chrome and Safari (not IE), but the nice thing is that if a user's browser does not support Service Worker then apps and websites which utilise Service Worker will still work, but will fall back to always using the network.

## Gotchas

Service Workers actually have quite a few gotchas, here are the ones which caused me the most confusion:

1) You must place your service worker in or above the directory of the files that you wish to cache, otherwise the Service Worker will not be able to find them.

2) The first time you load a particular page which has a service worker, the service worker will still not intercept network requests even after activation, this is because if a page was not loaded with a Service Worker then neither are its subresources.

