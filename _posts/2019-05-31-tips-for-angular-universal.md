---
layout: post
title: "Getting start with Angular Universal, and avoiding the pitfalls"
image: ""
published: false
versions: Angular 8.x.x
---

Angular Universal is a technology which allows you to create Angular apps which render both in the browser and on a Node.js server. 

Unlike a normal browser rendered Angular app (Single Page Application), a Universal app is indexable by search engines and social media crawlers, allowing it able to reap the benefits of SEO.

In this article, I will talk about when you should use Angular Universal, how it works and how to get started. Finally, I will discuss how you can avoid the most common pitfalls, and the adaptions you must make to your codebase in order to facilitate Universal rendering.

# When to use Angular Universal

There are [3 main reasons](https://angular.io/guide/universal#why-use-server-side-rendering) to use Angular Universal:

1. **To facilitate crawlers from Search Engines and social media sites to crawl your application.**

    In reality, this is by far the biggest reason to use Angular Universal. 
    
    It will allow your application to be indexed by search engines such as Google, and allow social media sites to provide a preview of a page of your application when someone posts a link to it. A normal client rendered Angular application will either not be indexed by search engines or have very poor SEO.

    At the time of writing (01/06/19), some crawlers such as the Google crawler can execute JavaScript. However, [in practice, people have had mixed results](https://seopressor.com/blog/javascript-seo-how-does-google-crawl-javascript/) in actually getting their client rendered applications crawled and indexed by Google. Therefore, if you are building an Angular app and SEO is a big concern for you, Angular Universal is your best option.

2. **To improve performance of devices which don't support JavaScript**

    These devices will be unable to run a browser rendered Angular app. With Angular Universal, you provide a server rendered version of the page, without any JavaScript. Whether this no JavaScript version of the page is actually any use to the user will depend on the application in question.

3. **To decrease the (perceived) first page load time**

    With a browser rendered Angular app, when a user first navigates to the app they will have to wait for several hundred KB of JavaScript to download, and then wait for Angular to bootstrap, before they ever see your application.

    When a user first navigates to an Angular Universal app, the browser first receives a server rendered version of the page that the user navigated to. The user then momentarily sees this JavaScript free, non-interactive, version of the app, while the JavaScript for the Angular app downloads and bootstraps.

# How Angular Universal works

Lets first look at what happens when an ordinary Angular app loads, and then compare this to what happens with Angular Universal.

## Browser rendered application

The sequence of events through which the page is loaded when a user navigates to a browser rendered Angular application goes like this:

1. The user navigates to the application.

2. The server serves up `index.html`.

3. `index.html` is rendered in the DOM and looks something like this:

    ```html
    <html lang="en">
        <head>
            ...
            <link rel="stylesheet" href="styles.3ff695c00d717f2d2a11.css">
        </head>
        <body>
            <app-root></app-root>
            <script type="text/javascript" src="runtime.ec2944dd8b20ec099bf3.js"></script>
            <script type="text/javascript" src="polyfills.3bfd66addbd0d2814591.js"></script>
            <script type="text/javascript" src="main.8d3bca5df2cd7b3fd5cf.js"></script>
        </body>
    </html>
    ```

4. The JavaScript and CSS files referenced in `index.html`, which make up the Angular app, are downloaded.

5. The Angular application bootstraps and renders inside of `app-root`.

## Universal application

When a user navigates to an Angular Universal application, the sequence of events through which the page is loaded goes like this:

1. The user navigates to the application.

2. The server (a Node.js server) executes the application in order to render the page that the user navigated to, and then serves up `index.html`

3. `index.html` is rendered in the DOM and looks something like this:

    ```html
    <html lang="en">
        <head>
            ...
            <link rel="stylesheet" href="styles.3ff695c00d717f2d2a11.css">
            <style ng-transition="serverApp">
                ...
            </style>
        </head>
        <body>
            <app-root></app-root>
            <script type="text/javascript" src="runtime.ec2944dd8b20ec099bf3.js"></script>
            <script type="text/javascript" src="polyfills.3bfd66addbd0d2814591.js"></script>
            <script type="text/javascript" src="main.8d3bca5df2cd7b3fd5cf.js"></script>
        </body>
    </html>
    ```

    Notice that here, the HTML for the page has been rendered inside of the `app-root` element. Also, any styles needed for this page are also included inside the `style` tags in the page head. At this point no JavaScript has been downloaded or executed, so the page is 'non-interactive'.

4. The JavaScript and CSS files referenced in `index.html` are downloaded.

5. The Angular application bootstraps and renders inside of `app-root`, 'taking over from' the server rendered version.

# Getting started with Universal

It is [super easy](https://angular.io/guide/universal) to set up an Angular Universal application with the Angular CLI.

The first step is to create a new Angular application:

```
ng new universal-demo
cd universal-demo
```

You can turn this into an Angular Universal application with the following command:

```
ng add @nguniversal/express-engine --clientProject universal-demo
```

And then to build and serve your application with universal rendering:

```
npm run build:ssr && npm run serve:ssr
```

You have just created, built and served an Angular Universal application!

If we look at the codebase for this application, we see that Angular has made quite a few changes including the creation of some new files. We will take a look at these now.

## A deep dive into the code

Lets take a deeper look at the changes that were made to transform our Angular app into an Angular Universal app, and try to get an understanding of how it works.

There are 3 main changes which haved been made:

1. The addition of Node.js server code inside of `server.ts`.
2. Addition of build scripts in package.json and supporting files for these scripts.
3. Changes to the source code files in the existing Angular app.

Lets take a more detailed look at each of these in turn.

## Node.js server code

This is your `node.js` server code, it is where the server side rendering of the application takes place.

```ts
import 'zone.js/dist/zone-node';
import 'reflect-metadata';
import {enableProdMode} from '@angular/core';
// Express Engine
import {ngExpressEngine} from '@nguniversal/express-engine';
// Import module map for lazy loading
import {provideModuleMap} from '@nguniversal/module-map-ngfactory-loader';

import * as express from 'express';
import {join} from 'path';

// Faster server renders w/ Prod mode (dev mode never needed)
enableProdMode();

// Express server
const app = express();

const PORT = process.env.PORT || 4000;
const DIST_FOLDER = join(process.cwd(), 'dist/browser');

// * NOTE :: leave this as require() since this file is built Dynamically from webpack
const {AppServerModuleNgFactory, LAZY_MODULE_MAP} = require('./dist/server/main');

// Our Universal express-engine (found @ https://github.com/angular/universal/tree/master/modules/express-engine)
app.engine('html', ngExpressEngine({
  bootstrap: AppServerModuleNgFactory,
  providers: [
    provideModuleMap(LAZY_MODULE_MAP)
  ]
}));

app.set('view engine', 'html');
app.set('views', DIST_FOLDER);

// Example Express Rest API endpoints
// app.get('/api/**', (req, res) => { });
// Server static files from /browser
app.get('*.*', express.static(DIST_FOLDER, {
  maxAge: '1y'
}));

// All regular routes use the Universal engine
app.get('*', (req, res) => {
  res.render('index', { req });
});

// Start up the Node server
app.listen(PORT, () => {
  console.log(`Node Express server listening on http://localhost:${PORT}`);
});
```

When I first came across this file, as an Angular developer who deals with primarily frontend JavaScript and not Node.js and Express, it took me a bit of time and research to understand what was happening.

Lets break this down and look at it one piece at a time.

```ts
import 'zone.js/dist/zone-node';
import 'reflect-metadata';
import {enableProdMode} from '@angular/core';
// Express Engine
import {ngExpressEngine} from '@nguniversal/express-engine';
// Import module map for lazy loading
import {provideModuleMap} from '@nguniversal/module-map-ngfactory-loader';

import * as express from 'express';
import {join} from 'path';

// Faster server renders w/ Prod mode (dev mode never needed)
enableProdMode();

// Express server
const app = express();

const PORT = process.env.PORT || 4000;
const DIST_FOLDER = join(process.cwd(), 'dist/browser');

// * NOTE :: leave this as require() since this file is built Dynamically from webpack
const {AppServerModuleNgFactory, LAZY_MODULE_MAP} = require('./dist/server/main');
```

In this first section we:

* Import all of our dependencies.
* Call the `enableProdMode` function from `@angular/core`.
* Set the port for the server and the location of the dist folder. Here, `process.env` is the operating system environment - if the port is not specified, we default to 4000.
* Instantiates an Express server.


```ts
// Our Universal express-engine (found @ https://github.com/angular/universal/tree/master/modules/express-engine)
app.engine('html', ngExpressEngine({
  bootstrap: AppServerModuleNgFactory,
  providers: [
    provideModuleMap(LAZY_MODULE_MAP)
  ]
}));

app.set('view engine', 'html');
app.set('views', DIST_FOLDER);
```

In the above, the `app.engine` function call registers Angular's `ngExpressEngine` as the templating engine to use for `.html` files. We then tell Express to use this templating engine, and also tell Express where to find our views (ie. where to find `index.html`).

```ts
// Example Express Rest API endpoints
// app.get('/api/**', (req, res) => { });
// Server static files from /browser
app.get('*.*', express.static(DIST_FOLDER, {
  maxAge: '1y'
}));

// All regular routes use the Universal engine
app.get('*', (req, res) => {
  res.render('index', { req });
});

// Start up the Node server
app.listen(PORT, () => {
  console.log(`Node Express server listening on http://localhost:${PORT}`);
});
```

Here, we route get requests to the server.

1. **\*.\*** - urls with a single '.' in them, ie. `*.css`, `*.js`, `*.png` will be served up as static files.
2. **\*** - all other urls which did not get matched by the first path are rendered using the `ngExpressEngine`.

We then start the server using `app.listen` - it will listen for requests on our specified port.


## Build scripts

A few build scripts were added to `package.json`:

* `build:ssr` will build the application.
* `serve:ssr` will run a node server and serve the application with Universal Rendering.
* `build:client-and-server-bundles` is an intermediatory step which builds the application for the browser and server. You can see the server files which this outputs inside of `dist/server`, and the browser files in `dist/browser`.
* `compile:server` takes your `server.ts` server code and your application code, compiles them, and combines them into a single file, `server.js`. You can see this file in the root of your `dist` folder.

You will also see that some additional files `webpack.server.config.js`, `tsconfig.server.json` have been added in order to to facilitate these build scripts, as well as some modifications to `angular.json`.

## Changes to the application source

If we look at the application source, we see that 2 files were added:

1. `main.server.ts` is the server entry point for your application.
2. `app.server.module.ts` is the server application module.

And 2 other files were modified:

1. `app.module.ts`

    ```ts
        imports: [
            ...
            BrowserModule.withServerTransition({ appId: 'serverApp' })
            ...
        ]
    ```

    This allows Angular to remove any styles added during the server render, ie. the styles found here in `index.html`:

    ```html
    <style ng-transition="serverApp">
        ...
    </style>
    ```

2. `main.ts`

    ```ts
    document.addEventListener('DOMContentLoaded', () => {
    platformBrowserDynamic().bootstrapModule(AppModule)
    .catch(err => console.error(err));
    });
    ```

    The above waits for the DOM to fully load before rendering the Angular app in the browser, this is useful for Angular Universal state transfer, as we will see in the next section.

# State transfer

// TODO

## Pitfalls and adaptions

While for the most part, your application should run in the server just as it does in the browser, there are [a few gotchas](https://github.com/angular/universal/blob/master/docs/gotchas.md).

1. Avoid referencing `window`

    It does not exist in on the Node.js server.

2. Avoid manipulating the DOM via `nativeElement`. 

    `nativeElement` exposes the native DOM api, which does not exist on the Node.js server.

3. Memory leaks can be a showstopper on the Node.js server.

4. Avoid long running asynchronous operations during the page load where possible.

    Angular Universal will wait for these to finish before serving up `index.html`.

5. Set up your Express server to use appropriate middleware such as `compression`?

6. 

## Conclusions