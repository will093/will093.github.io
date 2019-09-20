---
layout: post
title: "Pure functions in JavaScript"
image: ""
published: false
---

Pure functions are a really great thing! You may have come across them before if you have ever been exposed to functional programming, and while they could be seen as the cornerstone on which functional programming is built, you don't need to know any fancy functional programming terminology or have any prior knowledge to understand and make use of them.

In this article, we will look at what a pure function is, why you would use them, and some examples of pure functions (and impure functions) in JavaScript.

# What is a pure function

Lets first look at a simple example to get an idea of what a pure function is, and then follow this up with a more formal definition.

## A simple example

The following is a pure function:

```js
function addThemNumberz(a, b) { 
  // Perform a calculation (or we could say a mapping) on the input values (arguments) and return the result.
  return a + b; 
}
```

The above is a pure function. The return value of a call to the above function is determined entirely by the arguments with which it is called, and it doesn't modify any variables outside of the function. This is in contrast to the following function:

```js
var selectedAge = 7;
function setAgeToSelected(person) {
  person.age = selectedAge;
}
```

The above is a great example of impure function, it probably could not be any less pure if it tried. It works by mutating the state of the object passed to it as an argument, and it sets the new state using the value of a variable external to the function.

## A formal definition

More formally, a pure function must satisfy the following 2 requirements:

1. **The return value depends only on the arguments passed in**

  Therefore, we can say that a pure function must not depend on the value of any variable external to the function.

2. **No side effects**

  A pure function does not do anything other than produce a return value by performing some calculation using the arguments passed in. It does not reassign non-local variables, mutate state, or call any non-pure functions.

The first rule is sometimes stated as 'No side causes'. I like this definition, and find that it helps me to break down any violations of function purity into two categories - 'side cause' and 'side effect. 

We will have a deeper look at these requirements and some examples later on, but first lets discuss why we would want to use pure functions in JavaScript.

# Why use pure functions in JavaScript?

* **An application which uses more pure functions is easier to reason about**

  A pure function does not cause any side effects, and so when we call a pure function we can be sure that we have not altered any state elsewhere in the program. Functions which modify non-local state are one of the biggest causes of spaghetti code, and using more pure functions can help you avoid this problem.

  // TODO: example
  ```js
  function createNonLocalStateMutationBugs() {
    // Modify some state in some other module

    // Use a mutating method on the array passed in.


  }
  ``` 

  It would be great if at the call site of a function we could know if it was pure or not. However, in JavaScript we cannot do this. The best we can do is have a convention where functions in particular locations within an application are kept pure (such as redux). I will talk more about this later.

* **They are easy to unit test.**

    A pure function is easier to test because we don't have any sources of side effects or side causes. This means no external dependencies to mock, and no need to test that some particular side effect worked as expected.

    Using `jasmine`, a unit test for a pure function becomes very simple, and in fact can often be expressed a one liner:
    
    ```js
    // Unit test for a pure function, 'addThemNumbers', which adds 2 number together.
    expect(addThemNumberz(24, 39)).toEqual(63);
    ```

* **Ability to leverage additional functional programming techniques.**

  Pure functions are the foundation of functional programming, and by learning to write pure functions, you will end up writing functions which are more reusable, and can be used with other functional programming techniques such as functional composition. I won't go into this in depth here as this article is specifically about pure functions and not any other aspect of functional programming.

* **Can be memoized.**

  Memoization is essentially a caching of the computed value of a function for a particular set of arguments the first time that the function is called with those arguments. If the function is called again with the same arguments, then the cached result is returned. Of course, if the function is not pure, then memoization cannot really take place as functions cannot be memoized.

  We could write a function for memoization of another function (TODO: link to this)

  TypeScript has a memoization pipe, which can automatically memoize a function.

* **Can be parrallelised**

  Since we typically run JavaScript in a single threaded environment this is not such a big advantage as compared to in some other languages. However, its worth noting that pure functions can safely be run in parralel with one another on separate threads.

The final reason to start using more pure functions in your JavaScript is that it doesn't have to be all or nothing to gain these advantages. In fact in the real world, an application is never entirely made up of pure functions, or it would have no observable output. 

Patterns like redux exist to help you organise your application so that pure functions go in one place, and impure functions (side effects) go in another. But even if you are not using such a pattern, just by consciously choosing to use pure functions over impure ones when you notice the opportunity, you will improve your programming technique.

# Pure and impure functions in JavaScript

We looked at a simple example of a pure function and an impure function already. But lets look at a few more examples and discuss why they are pure or impure.

## Pure functions

// TODO:

## Impure functions

// TODO:

# How to use more pure functions in JavaScript


# Conclusion

TODO:

