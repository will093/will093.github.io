---
layout: post
title: "Angular Universal for Angular developers"
image: "/images/angular-universal-for-angular/night-sky-1.jpg"
published: true
versions: Angular 8.x.x
---

Angular Universal is a technology which allows you to create Angular apps that render both in the browser and on a Node.js server. Unlike a normal browser rendered Angular app (single page application), a Universal app is indexable by search engines and social media crawlers, allowing it to reap the benefits of SEO.

This article assumes some experience working with Angular - it is aimed at Angular developers looking to learn about Angular Universal, and pairs nicely with my [Angular Universal gotchas](https://willtaylor.blog/angular-universal-gotchas) article.

In the article, I will discuss why you might use Angular Universal and what the alternatives are, how Universal works at a high level, and how to get started. I will then do a deep dive into the code that makes up an Angular Universal application. Finally, I will talk about transferring data from a server rendered Angular application to a client rendered application via the Angular `TransferState` module.

# Why use Angular Universal?

There are [3 main reasons](https://angular.io/guide/universal#why-use-server-side-rendering) to use Angular Universal:

1. **To facilitate crawlers from search engines and social media sites.**

    In reality, this is by far the biggest reason to use Angular Universal. 
    
    It will allow your application to be indexed by search engines such as Google, and allow social media sites to provide a preview of a page of your application when someone posts a link to it. A normal client rendered Angular application will either not be indexed by search engines or have very poor SEO.

    At the time of writing (5th July 2019), some crawlers such as the Google crawler can execute JavaScript. However, [in practice, people have had mixed results](https://seopressor.com/blog/javascript-seo-how-does-google-crawl-javascript/) in actually getting their client rendered applications crawled and indexed by Google. Therefore, if you are building an Angular app and SEO is a big concern for you, Angular Universal is a good option.

2. **To improve performance of devices which don't support JavaScript**

    These devices will be unable to run a browser rendered Angular app. With Angular Universal, you provide a server rendered version of the page, without any JavaScript. Whether this 'JavaScript free' version of the page is actually any use to the user will depend on the application in question.

3. **To decrease the (perceived) first page load time**

    With a browser rendered Angular app, when a user first navigates to the app they will have to wait for several hundred KB of JavaScript to download, and then wait for Angular to bootstrap, before they ever see your application.

    When a user first navigates to an Angular Universal app, the browser first receives a server rendered version of the page that the user navigated to. The user then momentarily sees this JavaScript free, non-interactive version of the app, while the JavaScript for the Angular app downloads and bootstraps.

At this point it is worth touching on another solution - prerendering - which achieves the same thing as universal rendering, but in a different way.

## Prerendering

Prerendering aims to solve the same problem as Angular Universal. 

In a nutshell, prerendering also serves up a server rendered version of your site which the client rendered version will then 'take over from'. How prerendering differs from universal rendering is that each page/route of your site is rendered at build time rather than run time. 

Prerendering has both advantages and disadvantages compared to universal rendering. 

The **advantages** are:

* Prerendering removes the overhead of running a Node.js server in your production environment.
* Prerendering will result in a faster page load than universal rendering. This is because at runtime, all the pages of your site have already been rendered and simply need to be served as static files.

The **disadvantages** are:

* You must rebuild and deploy your site any time that any content changes.
* Prerendering generally won't work for sites with route parameters.

For sites which do not regularly update their content and do not have any 'dynamic' route parameters, prerendering is usually the simpler and better solution. 

I will not discuss prerendering any further in this article, but I recommend [this tutorial](https://medium.com/@maciejtreder/prerender-angular-application-be-seo-ae1183c621cb) for getting started with prerendering in Angular.

# How Angular Universal works

Lets first look at what happens when an Angular single page application loads, and then compare this to what happens with Angular Universal.

## Angular single page application

The sequence of events through which the page is loaded when a user navigates to an Angular single page application goes like this:

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

5. The Angular application bootstraps in the browser and renders inside of `app-root`.

## Angular Universal application

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
                [_nghost-serverApp-c0]{display:block}
                ...
            </style>
        </head>
        <body>
            <app-root>
                <div><h1>My application</h1></div>
                ...
            </app-root>
            <script type="text/javascript" src="runtime.ec2944dd8b20ec099bf3.js"></script>
            <script type="text/javascript" src="polyfills.3bfd66addbd0d2814591.js"></script>
            <script type="text/javascript" src="main.8d3bca5df2cd7b3fd5cf.js"></script>
        </body>
    </html>
    ```

    Notice that here, the HTML for the page has been rendered inside of the `app-root` element. Also, any styles needed for this page are included inside of the `style` tags in the page head. At this point no JavaScript has been downloaded or executed, so the page is 'non-interactive'.

4. The JavaScript and CSS files referenced in `index.html` are downloaded.

5. The Angular application bootstraps in the browser and renders inside of `app-root`, and takes over from the server rendered version of the app.

# Getting started with Universal

It is super easy to set up an Angular Universal application with the Angular CLI.

For the purposes of this article you can clone the following repo, containing an Angular single page application which we will later transform into an Angular Universal application.

```bash
git clone git@github.com:will093/universal-ghibli-demo.git
```

If you build and serve this application, you will see that what we have is an application which requests a collection of Studio Ghibli films from the [Ghibli api](https://ghibliapi.herokuapp.com/) and then sorts and displays them. Lets get to work transforming this into a Universal app!

Angular provides tools for transforming a normal Angular application into a Universal app, using the following command:

```bash
ng add @nguniversal/express-engine --clientProject universal-ghibli-demo
```

And then to build and serve the application with universal rendering:

```bash
npm run build:ssr && npm run serve:ssr
```

Well done! You have just created, built and served an Angular Universal application!

![Ghibli demo application](/images/angular-universal-for-angular/ghibli-demo-application.jpg)

If we look at the codebase for this application, we see that Angular has made quite a few changes in order to transform it into a Universal app - we will take a look at these now.

# A deep dive into an Angular Universal app

Lets take a deeper look at the changes that were made to transform our Angular single page application into an Angular Universal app. Conceptually, we can divide up the changes which have been made into 3 areas:

1. The addition of Node.js server code.
2. Addition of build scripts in package.json and supporting files for these scripts.
3. Changes to the source code files in the existing Angular app.

Lets take a more detailed look at each of these in turn.

## 1) Addition of Node.js server code

Inside of `server.ts` is is your Node server code, it is where the server side rendering of the application takes place.

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

// Our Universal express-engine 
// (found @ https://github.com/angular/universal/tree/master/modules/express-engine)
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

When I first came across this file, as an Angular developer who deals with primarily frontend JavaScript and not Node or Express, it took me a bit of time and research to understand what was happening.

Lets break this down and look at it one section at a time.

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
* Instantiate an Express server.

On to the next section:

```ts
// Our Universal express-engine 
// (found @ https://github.com/angular/universal/tree/master/modules/express-engine)
app.engine('html', ngExpressEngine({
  bootstrap: AppServerModuleNgFactory,
  providers: [
    provideModuleMap(LAZY_MODULE_MAP)
  ]
}));

app.set('view engine', 'html');
app.set('views', DIST_FOLDER);
```

In the above, the `app.engine` function call registers Angular's `ngExpressEngine` as the templating engine to use for `.html` files. 

We then tell Express to use this templating engine, and also tell Express where to find our views (ie. where to find `index.html`).

Now lets look at the final section:

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

Here, we handle the routing for GET requests made to the server.

1. **\*.\*** - urls with a single '.' in them, ie. `*.css`, `*.js`, `*.png` will be served up as static files.
2. **\*** - all other urls which did not get matched by the above are rendered using the `ngExpressEngine`.

We then start the server using `app.listen` - it will listen for requests on our specified port.


## 2) Addition of build scripts

A few build scripts were added to `package.json`:

* `build:ssr` will build the application.
* `serve:ssr` will run a node server and serve the application with Universal Rendering.
* `build:client-and-server-bundles` is an intermediatory step which builds the application for the browser and server. You can see the server files which this outputs inside of `dist/server`, and the browser files in `dist/browser`.
* `compile:server` takes your `server.ts` server code and your application code, compiles them, and combines them into a single file, `server.js`. You can see this file in the root of your `dist` folder.

You will also see that some additional files `webpack.server.config.js`, `tsconfig.server.json` have been added in order to to facilitate these build scripts, as well as some modifications to `angular.json`.

## 3) Changes to the application source code

If we look at the application source, we see that 2 files were added:

1. `main.server.ts` is the server entry point for your application.
2. `app.server.module.ts` is the server application module.

Also, 2 files were modified:

1. `app.module.ts`

    ```ts
        imports: [
            ...
            BrowserModule.withServerTransition({ appId: 'serverApp' })
            ...
        ]
    ```

    This allows Angular to remove any styles added during the server render, ie. the styles found in `index.html`:

    ```html
    <style ng-transition="serverApp">
        ...
    </style>
    ```

2. `main.ts`

    ```ts
    document.addEventListener('DOMContentLoaded', () => {
        platformBrowserDynamic()
            .bootstrapModule(AppModule)
            .catch(err => console.error(err));
    });
    ```

    The above waits for the DOM to fully load before rendering the Angular app in the browser. This is necessary for the Angular TransferState module (which we will discuss in the next section) to work correctly.

# Transfer State

In the case of a Universal application which makes requests to an API in order to render the page, there are a couple of problems. 

* Each API request will be unnecessarily made twice, both on the client and the server.
* The user may see some flickering as the page loads. This is because they will initially see the server rendered app, then when the client app takes over it needs to wait for a response from the server before rendering the page.

The `TransferState` module provides a solution for transferring application state from the server to the client. From the Angular docs:

> TransferState - A key value store which gets transferred from the application on the server side to the application on the client side.

We can use this module to store responses from http requests - we give each response a key, and then retrieve it on the client. 

Lets demonstrate this by adding the `TransferState` module to our demo application that we created earlier.

## Implementing Transfer State

If you watch the page load of our Angular Universal Ghibli app, you should see some flickering as described in the previous section. We will now implement a solution for transfering state from the server to the client in our Ghibli films demo application.

In `app.module.ts`:

```ts
...
import { BrowserModule, BrowserTransferStateModule } from '@angular/platform-browser';
...

imports: [
    ...
    BrowserTransferStateModule,
    ...
],
```

In `app.server.ts`:

```ts
...
import { ServerModule, ServerTransferStateModule } from '@angular/platform-server';
...

imports: [
    ...
    ServerTransferStateModule,
    ...
],
```

Next, lets create an interceptor, which will intercept HTTP requests and store them using `TransferState` as necessary.

We add the following inside of `src/app/interceptors/transfer-state.interceptor.ts`:

```ts
import { HttpEvent, HttpHandler, HttpInterceptor, HttpRequest, HttpResponse } from '@angular/common/http';
import { Inject, Injectable, PLATFORM_ID } from '@angular/core';
import { Observable, of } from 'rxjs';
import { StateKey, TransferState, makeStateKey } from '@angular/platform-browser';
import { isPlatformBrowser, isPlatformServer } from '@angular/common';
import { tap } from 'rxjs/operators';

@Injectable()
export class TransferStateInterceptor implements HttpInterceptor {

  constructor(
    private transferState: TransferState,
    @Inject(PLATFORM_ID) private platformId: any,
  ) {}

  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

    // For this demo application, we will only worry about State Transfer for get requests.
    if (request.method !== 'GET') {
      return next.handle(request);
    }


    // Use the request url as the key.
    const stateKey: StateKey<string> = makeStateKey<string>(request.url);

    // For any http requests made on the server, store the response in State Transfer.
    if (isPlatformServer(this.platformId)) {
      return next.handle(request).pipe(
        tap((event: HttpResponse<any>) => {
          this.transferState.set(stateKey, event.body);
        })
      );
    }

    // For any http requests made in the browser, first check State Transfer for a 
    // response corresponding to the request url.
    if (isPlatformBrowser(this.platformId)) {
      const transferStateResponse = this.transferState.get<any>(stateKey, null);
      if (transferStateResponse) {
        const response = new HttpResponse({ body: transferStateResponse, status: 200 });

        // Remove the response from state transfer, so any future requests to 
        // the same url go to the network (this avoids us creating an 
        // implicit/unintentional caching mechanism).
        this.transferState.remove(stateKey);
        return of(response);
      } else {
        return next.handle(request);
      }
    }
  }
}
```

Now if we rebuild and run the app, we see that the flicker on the initial page load is gone. If we look in the network tab then we can see that the browser no longer makes a request to the Ghibli api - excellent!

But how does this work? How does the server transfer this state data to the client?

If in the browser we go to the network tab and look inside of the `index.html` which gets served up for our application, we see the following at the bottom of the file:

```html
<script id="serverApp-state" type="application/json">{&q;https://ghibliapi.herokuapp.com/films&q;:[{&q;id&q;:&q;4e236f34-b981-41c3-8c65-f8c9000b94e7&q;,&q;title&q;:&q;Only Yesterday&q...</script>
```

We can see here that `TransferState` stores the application state data inside of a script which gets appended to the end of `index.html` during the server render!

## Conclusions

In this article, we went through the reasons why you might use Angular Universal, what the alternatives are, and how Angular Universal works at a high level. 

We then looked at how to get started with Universal by transforming an existing Angular application into an Angular Universal app, and we did a deep dive into the code which gets generated when we create a Universal app with the Angular CLI tools. 

Finally we went through how to add the Angular `TransferState` module into an application, and implemented an interceptor to handle state transfer functionality.

The code for the completed Angular Universal Ghibli Demo application can be found [here](https://github.com/will093/universal-ghibli-demo/tree/universal-app).

If you found this article useful, I recommend also reading my [Angular Universal gotchas](https://willtaylor.blog/angular-universal-gotchas) article.

# Helpful Resources

The following were really helpful for writing this article, and I very much recommend them as further reading:

* <https://blog.angular-university.io/angular-universal/>
* <https://www.twilio.com/blog/faster-javascript-web-apps-angular-universal-transferstate-api-watchdog>