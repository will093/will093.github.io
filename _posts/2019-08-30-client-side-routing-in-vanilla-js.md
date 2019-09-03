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

# Implementing routing

- Removing URL hash

# Additional features

- Nested routes
- Parameterised routes
- Route data
- Combining with a view binding system

# Conclusion