---
layout: post
title: "Implementing client side routing in VanillsJS"
image: ""
published: true
---

In contrast to traditional server side routing, SPA frameworks such as Angular and React use client side routing to handle navigation around an application. This client side routing is usually handled by some routing module or package, and for many developers how this routing actually works is something of a mystery. 

In this article I will try and explain how client side routing works and walk you through how you can implement this client side routing functionality yourself. 

Also, let me state that more than anything else this article is just for learning purposes, most of the time you are best off just using the router package or module which comes with your framework of choice. 

# Routing basics

First lets think about the behaviour that we have grown to expect from a single page application.

* When the application is first loaded, the route which is loaded and hence the content which is displayed is based on the url.
* When the user navigates around within the application, the url changes. Without a full page reload a new route and hence some new content is displayed on the page, based on this new url.
* When the user navigates using the browser forward/back buttons, they move through the stack of routes that they previously visited.

When we think about implementing this ourselves, there are two main technical challenges which we are confronted with:

1. How do we change a url without causing a page reload, and preserve the behaviour of the browser forward/back buttons?
2. How do we map any given URL to the appropriate template (or component)?

The answer to question 2 is more complicated and depends on exact requirements - we will look at implementing an algorithm which can do this later on. As for changing a url without causing a page reload, we will use a method on the `history` api called `pushState`.

## history.pushState

The [history.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History_API#The_pushState()_method) method is part of HTML5 and is supported by all modern browsers. It allows you to push an entry to the browser's `history` object, specifying a state object for passing data to the new history entry, a title, and the url for the history entry.

```js
window.history.pushState({ data: 'some data' },"Page title", '/some-path');
```

We can test out the above by pasting it into the console inside of Chrome's developer tools. You should see that the url changes from whatever it was to '\<base-url\>/some-path'.

Now, if you type the command `window.history` into the console, you will see that the history object has been updated with this new entry, including the state object which we passed it, and the title (note that confusingly this [title will not be used as the page title](https://stackoverflow.com/a/13817807/11047122) in most browsers, and is just used as an 'advisory' title for this history entry).

![Chrome developer tools history object](/images/client-side-routing-vanilla-js/pushstate-console.png)

Also, note that we can navigate backwards and forwards from this '/some-url' using the browser navigation buttons.

Now that we have had a look at routing basics including `history.pushState`, we are ready to implement a simple Vanilla JS application with basic routing.

# Implementing a simple app

Our 'app' will just consist of an `index.html` containing the following code:

```html
<html>

<head>
  <title>Client Side Routing Demo</title>
</head>

<body>
  <header>
    <ul>
      <li><button>Home</button></li>
      <li><button>About</button></li>
      <li><button>Contact</button></li>
    </ul>
  </header>
  <div>
    <h1>Hello!</h1>
  </div>
</body>

</html>
```

Ok, so its a pretty basic application - in fact just a HTML template.

To run this, we will use an npm package called `http-server-spa`. This package is similar to `http-server`, but will allow us to fall back to `index.html` if a route does not match any resource on the server. This behaviour is the usual behaviour of a single page application, and will be useful for the purpose of this tutorial.

```
npm install http-server-spa -g
```

Now to run our application, from the root of your project, run:

```
http-server-spa . ./index.html
```

This will server the current folder, with `index.html` as a fallback.

If you navigate to `http://localhost:8080`, you should see the page/application.

# Implementing routing

First lets create a `routes.js` file which will contain the routes for our application.

```js
const routes = [
  {
    path: '/',
    template: '<h1>Home</h1>'
  },
  {
    path: '/about',
    template: '<h1>About</h1>',
  },
  {
    path: '/contact',
    template: '<h1>Contact</h1>',
  },
];
```

This is just an array of route objects, our matching algorithm will loop through this array and attempt to match the URL to one of these. We won't worry about more advanced features like nested routes or parameterised routes just yet.

Now lets implement our router - create a file named `router.js` and add the following code:

```js
class Router {

  constructor(routes) {
    this.routes = routes;
    this._loadInitialRoute();
  }

  loadRoute(...urlSegments) {
    // Attempt to match the url to a route.
    const matchedRoute = this._matchUrlToRoute(urlSegments);

    // Push a history entry with the new url.
    // We pass an empty object and an empty string as the historyState and title arguments, but their values do not really matter here.
    const url = `/${urlSegments.join('/')}`;
    history.pushState({}, '', url);

    // Append the given template to the DOM inside the router outlet.
    const routerOutletElement = document.querySelectorAll('[data-router-outlet]')[0];
    routerOutletElement.innerHTML = '';
    // insertAdjacentHtml preserves any event handlers (setting innerHTML does not).
    routerOutletElement.insertAdjacentHTML('beforeend', matchedRoute.template);
  }

  _matchUrlToRoute(urlSegments) {
    // Try and match the URL to a route.
    const matchedRoute = this.routes.find(route => {
      // Assume that the route path always starts with a slash, and so the first item in the segments array 
      // will always be an empty string. Slice the array to ignore this empty string.
      const routePathSegments = route.path.split('/').slice(1);
      // If there are different numbers of segments, then the route does not match the URL.
      if (routePathSegments.length !== urlSegments.length) {
        return false;
      }

      // If each segment in the url matches the corresponding route path, then the route is matched.
      return routePathSegments.every((routePathSegment, i) => routePathSegment === urlSegments[i]);
    });
    return matchedRoute;
  }

  _loadInitialRoute() {
    // Figure out the path segments for the route which should load initially.
    const pathnameSplit = window.location.pathname.split('/');
    const pathSegments = pathnameSplit.length > 1 ? pathnameSplit.slice(1) : '';
    // Load the initial route.
    this.loadRoute(...pathSegments );
  }
}
```

In the above, `loadRoute` has 3 main steps:

1. Attempt to match the url to a route. 

  Our `_matchUrlToRoute` function will handle this, by searching for the first route where there are equal number of segments in the route path, and for each index the segments at that index are identical.

2. Build a url from the `urlSegments`, and push a history entry with that url (this will update the url shown in the browser address bar).

3. Replace the contents of the element marked as `data-router-outlet` with the template corresponding to the matched route.

Finally lets bring all of this together inside of `index.js`, where we will create an instance of our router, passing to it our array of routes:

```js
// Initialise an instance of our router class.
const router = new Router(routes);
```

Our `index.html` also needs updating with our new routing functionality.

```html
<html>

<head>
  <title>Client Side Routing Demo</title>
</head>

<body>
  <header>
    <ul>
      <li><button onclick="router.loadRoute('')">Home</button></li>
      <li><button onclick="router.loadRoute('about')">About</button></li>
      <li><button onclick="router.loadRoute('contact')">Contact</button></li>
    </ul>
  </header>
  <div data-router-outlet>

  </div>
  <script src="../js/router.js"></script>
  <script src="../js/routes.js"></script>
  <script src="../js/index.js"></script>
</body>

</html>
```

Note how when our navigation buttons are clicked, we load the corresponding route by calling our `router.loadRoute` function, passing the url segments as individual string parameters.

Now if you run the application again and go back to your browser, you should be able to navigate between the routes that we have created using Home/About/Contact buttons. If you have a play around, you will also see that the browser back, forward and refresh buttons work as usual.

# Additional features

The router that we have implemented is very basic. Most client side routers support more advanced features such as nested routing/child routes, parameterised routes, route data resolvers, and more. We won't look at all of these in this article, but the one I will cover here is extending the router to work with parameterised routes.

## Implementing parameterised routes

We will update our router to support parameterised routes, using the ':' token to indicate a parameterised route segment. For example: `/products/:productId`.

First lets add a route of this form. Inside of `routes.js` add the following at the end of the routes array:

```js
const routes = [
  ...
  {
    path: '/products/:productId',
    template: `<h1>Product</h1>`,
  },
];
```

Once we have parameterised routes, it would be nice to be able to render the template based on our route parameters. Lets get rid of our template property and replace it with a `getTemplate` function which will take our route parameters as an argument.

```js
const routes = [
  {
    path: '/',
    getTemplate: (params) => '<h1>Home</h1>'
  },
  {
    path: '/about',
    getTemplate: (params) => '<h1>About</h1>',
  },
  {
    path: '/contact',
    getTemplate: (params) => '<h1>Contact</h1>',
  },
  {
    path: '/products/:productId',
    getTemplate: (params) => `<h1>Product ${params.productId}</h1>`,
  },
];
```

Next, we need to update our algorithm for matching a url to a route.

```js
_matchUrlToRoute(urlSegments) {
  // Try and match the URL to a route.
  const routeParams = {};
  const matchedRoute = this.routes.find(route => {
    // Assume that the route path always starts with a slash, and so the first item in the segments array 
    // will always be an empty string. Slice the array to ignore this empty string.
    const routePathSegments = route.path.split('/').slice(1);
    // If there are different numbers of segments, then the route does not match the URL.
    if (routePathSegments.length !== urlSegments.length) {
      return false;
    }

    // If each segment in the url matches the corresponding route path, then the route is matched.
    const match = routePathSegments.every((routePathSegment, i) => {
      return routePathSegment === urlSegments[i] || routePathSegment[0] === ':';
    });

    // If the route matches the url, pull out any params from the URL.
    if (match) {
      routePathSegments.forEach((segment, i) => {
        if (segment[0] === ':') {
          const propName = segment.slice(1);
          routeParams[propName] = decodeURIComponent(urlSegments[i]);
        }
      });
    }
    return match;
  });
  return { ...matchedRoute, params: routeParams };
}
```

Here, we have changed the logic of our matching algorithm. Now, for a url to match a route path, each segment of the route path must either match the url segment with the same index, or it can start with a `':'`, which signifies that it is a parametersied route parameter, and so will take the value of the url segment with the same index.

After a route is matched, we pull out the route parameters into an object. The key for each parameter is the value following the `':'` in the parameterised route segment, and the value is the url segment with the same index as that parameterised route segment.

We also need to update the last line of our `loadRoute` function, in order to wire everything up and render the templates for each route with its route params.

```js
loadRoute(...urlSegments) {
  ...
  routerOutletElement.insertAdjacentHTML('beforeend', matchedRoute.getTemplate(matchedRoute.params));
}
```

Finally, we will add some product buttons inside of `index.html` in order to test out our parameterised route:

```html
  <header>
    <ul>
      ....
      <li><button onclick="router.loadRoute('products', '1')">Product 1</button></li>
      <li><button onclick="router.loadRoute('products', '2')">Product 2</button></li>
    </ul>
  </header>
```

Now if you run the application again, you should see that these new product buttons load our parameterised route. If you type in a url of the form `/products/x` into the address bar, the string `'Product x'` will display in the page heading.

The above statement actually has some pretty serious security implications which i will touch on here.

## Shortcomings and security flaws

As I have already mentioned, this is not intended to be a production grade router. Apart from the fact that it is missing a number of features, it is vulnerable to JavaScript injection and XSS attacks.

We can easily illustrate how a user can inject JavaScript into our application through our parameterised route, and carry out an XSS attack. Lets examine the following HTML.

<img src="empty" onerror="alert('Damn our router is rubbish');"/>

`insertAdjacentHTML` won't execute any scripts that we include in a HTML string, so that is out of the question. However, we can attach event handlers to other HTML elements in order to execute JavaScript.

If we encode this and put the following URL into the address bar, lets see what happens.

![Router XSS](/images/client-side-routing-vanilla-js/router-xss.png)

Wow! That is not good. A production grade router would have sanitised and potential sources of JavaScript injection such as parameterised routes. 


# Conclusion



TODO: talk about hyperapp routing
