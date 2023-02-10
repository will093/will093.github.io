---
layout: post
title: ""
image: ""
published: false
---

# Error Handling

Every time you execute code and there is the possibility of an error you need to consider error handling.

## Why/where do errors usually occur

There are 2 main types of error that we must deal with.

* HTTP/network errors - problems with external services, or network connectivity can cause various HTTP errors.
* Bad Logic - buggy logic which handles data incorrectly or fails to consider edge cases.

Both of these are inevitable, as we can't prevent issues with external services from occuring, and bugs in logic are not always spotted before going to production..

## Error handling

In all cases, its best to have a dedicated error logging function, rather than logging errors directly to the console.

```js
function logError(error) {
  console.error(error);

  // Here we can conditionally send the error to an error logging service such as sentry, reformat the error etc...
}
```

### HTTP/network errors

Any time we write code that makes a HTTP request or accesses a resource over the network, we have to consider what should happen if it fails.

1) Display an error page

If some critical piece of data cannot be loaded during the preload of a page we usually want to display an error page.

// TODO: example of SWRStore preload resource
```html
<script>

</script>
```



2) Log the error and continue (for some non-essential part of the page)


```js
let data;
try {
  data = await fetchTheData();
} catch (e) {
  logError(e);
}
```

```js
let data;

const loadMyData = () => {
  data = loadResource("/resources/my-resource");
}

$: {
  if ($data?.error) {
    logError($data?.error)
  }
} 
```
2) Log the error and display some error UI to the user (for something that can confuse a user such as an error loading a the cart)


* Don't handle the error (bad idea?)

### Bad logic

A function which contains bugs/doesn't consider edge cases can cause errors. Similarly if data passed to the function is not the structure we expect then errors can occur. We should avoid writing brittle logic that doesn't consider edge cases... We want our code to not crash our site in production.

1) Error boundary (doesn't work for updates)
2) Optional chaining/null checks

## Errors in async functions and not crashing your server

## Error handling in crown

Should we have an error handler built into crown? When you call load resource or preload resource in could call the error handler when there is an error.