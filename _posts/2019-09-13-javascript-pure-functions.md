---
layout: post
title: "Pure functions in JavaScript"
image: "/images/pure-functions-in-js/cupcake-machine.jpg"
published: false
---

Pure functions are a really great thing! 

By understanding them and using them in the right way, your code will become more readable, testable, performant, and easier to reason about.

While pure functions can be viewed as the building blocks with which the functional programming paradigm is built, you don't need to know any fancy functional programming terminology or have any prior functional programming knowledge to understand and make use of them.

In this article we will look at what a pure function is, why you would want to use pure functions, and some examples of pure functions (and impure functions) in JavaScript.

# What is a pure function?

First, lets look at a simple example to get an idea of what a pure function is, and then follow this up with a more formal definition.

## A simple example

The following is an example of a pure function:

```js
function add(a, b) { 
  // Perform a calculation (or we could say a mapping) 
  // on the inputs (arguments) and return the result.
  return a + b; 
}
```

The reason this is a pure function is that the return value of a call to this function is determined entirely by the arguments with which it is called, and it doesn't modify any non-local state. 

Note that by non-local state, I mean anything other than state stored in local variables which are declared within the function - eg. `window`, `this` and variables declared outside the scope of the function are all non-local state.

In contrast, take a look at the following function:

```js
var selectedAge = 7;
function setAgeToSelected(person) {
  person.age = selectedAge;
}
```

This is a great example of impure function, it probably could not be any less pure if it tried! 

1. It is impure because it mutates the state of the `person` object passed to it as an argument.

2. It is impure because it sets `person.age` to the value of a variable external to the function.

## A formal definition

More formally, a pure function must satisfy the following 2 requirements:

1. **The return value depends only on the arguments passed in.**

    A pure function must not depend on the value of any variable external to the function.

2. **No side effects.**

    A pure function does not do anything other than produce a return value by performing some calculation using the arguments passed in. It does not reassign non-local variables, mutate state, or call any non-pure functions.

The first rule is sometimes stated as 'No side causes'. I like this definition, and find that it helps me to break down any violations of function purity into two categories - side cause and side effect. 

We will have a deeper look at these requirements and some more examples later on, but first lets discuss why we would want to use pure functions in JavaScript.

# Why use pure functions in JavaScript?

We have talked about what a pure function is, now lets talk about the reasons why you would use pure functions in JavaScript.

1. **An application which uses more pure functions is easier to reason about**

    A pure function does not cause any side effects, and so when we call a pure function we can be sure that we have not altered any state elsewhere in the program. Functions which modify non-local state are one of the biggest causes of spaghetti code, and using more pure functions can help you avoid this problem.

    Just by thinking a bit more about function purity and keeping functions pure where possible you can avoid code like this:

    ```js
    function createNonLocalStateMutationBugs(someArray) {
        // Modify some global state
        window.someVariable = 'new value';
        // Use a mutating method on the array passed in.
        someArray.reverse();
    }
    ``` 

    Okay, an extreme example - but I expect you have worked on a project where code not too dissimilar to this has caused issues.

    The caveat to this point is that at the call site of a function we do not know if it is pure or not, and for pure functions to make our code easier to reason about, we need to know this. In JavaScript, the best we can do is have some kind of methodology or framework whereby particular functions within an application are kept pure by convention (such as redux). I will talk more about this later.

2. **They are easy to unit test.**

    A pure function is easier to test because we don't have any sources of side effects or side causes. This means no external dependencies to mock, and no need to test that some particular side effect worked as expected.

    Using `jasmine`, a unit test for a pure function becomes very simple, and in fact can often be expressed a one liner:
    
    ```js
    // Unit test for a pure function, 'add', which adds 2 number together.
    expect(add(24, 39)).toEqual(63);
    ```

3. **Ability to leverage additional functional programming techniques.**

    Pure functions are the foundation of functional programming, and by learning to write pure functions, you will end up writing functions which are more reusable, and can be used with other functional programming techniques such as functional composition. I won't go into this in depth here as this article is specifically about pure functions and not any other aspect of functional programming.

4. **Pure functions can be memoized.**

    Memoization is essentially a caching of the computed value of a function for a particular set of arguments the first time that the function is called with those arguments. If the function is called again with the same arguments, then the cached result is returned.

    This can prove very useful in practice for performance reasons, and we can [write a memoize function](https://www.freecodecamp.org/news/understanding-memoize-in-javascript-51d07d19430e/) which can be used to memoize any other (pure) function.

5. **Can be parrallelised**

    Since we typically run JavaScript in a single threaded environment this is not such a big advantage. However, its worth noting that pure functions can safely be run in parralel with one another on separate threads.

# Examples of pure and impure functions in JavaScript

Earlier we looked at a simple example of both a pure function and an impure function. Now lets look at a few more examples and discuss *why* they are pure or impure.

## Examples of impure functions

There are a few common ways in which a function can be impure in JavaScript, lets take a look at each of these in turn:

1. **It mutates non-local state directly.**

    ```js
    function appendToArray(myArray, item) {
        window.myArray = [ ...myArray, item ];
        return window.myArray
    }
    ```

    This is a pretty simple example of a side effect. The function is impure because it modifies the global window object.

2. **It mutates it's arguments.**

    ```js
    function appendToArray(myArray, item) {
        return myArray.push(item);
    }
    ```

    This function may look pure at first glance, but in fact it has side effects. The array `push` method mutates the `myArray` argument - this non-local state mutation is very common and is often a cause of bugs.

3. **It mutates non-local state via it's `this` binding.**

    ```js
    let adder = {
        result: undefined,
        add: function(a, b) {
            this.result = a + b;
            return this.result;
        },
    };
    ```

    This is another example of a function with side effects. Here, our `add` function modifies non-local state via its `this` binding.

4. **It's return value depends on non-local state.**

    ```js
    let word = 'hello ';
    function appendToWord(value) {
        return `${word}${value}`
    }
    ```

    This is an example of a side cause. The return value depends on the value of the non-local `word` variable.

5. **It's return value depends on its `this` binding**

    ```js
    let appender = {
        word: 'hello ',
        appendToWord: function(value) {
            return `${this.word}${value}`;
        },
    };
    ```

    This is another example of a side cause. The return value of this function depends on its `this` binding.


## Examples of pure functions

We have look at several examples of impure functions. Now lets look at some examples of pure functions:

```js
function add(a, b) {
    return a + b;
}

function square(x) {
    return x * x;
}

function addThenSquare(a, b) {
    const addResult = add(a,b);
    return square(addResult);
}
```

All of the above functions are pure. `add` and `square` clearly do not have any side effects or side causes. `addThenSquare` is essentially just composed of these 2 pure functions, so it is also pure itself.

# How and when to use pure functions in JavaScript?

Now that you understand what a pure function is, and the benefits of using pure functions, the question is how and when to use them in your code.

Clearly, not every function you write can or should be pure. It is usually best to organise your application in such a way that particular parts of your application handle side effects, while other parts are made up of pure functions.

The most important thing is to be aware whenever you write a function of whether it is pure or not. By recognising this you will then be in a place to decide if the function would benefit from being pure, and if so, to ask yourself the right questions in order to make sure the function pure. 

* If the function mutates non-local state, can the state mutation instead be done by the caller, after the function call?

* If the function's return value depends on a non-local variable, can I instead pass that variable to the function as an argument?

As a final point, patterns such as redux offer a way to structure an application in a way that certain parts of the codebase (reducers and selectors) must only contain pure functions, while other parts may contain side effects/impure functions.

# Conclusion

We have looked at what a pure function is, the benefits of using pure functions in your code, and some examples of pure and impure functions. Finally, we looked an how and when to use pure functions. 

You should now be at a point where you can utilise pure functions in order to write cleaner, more readable, more performant code. An understanding of pure functions is also a great starting point for learning functional programming!

# Helpful Resources

The following were really helpful for writing this article, and I very much recommend them as further reading:

* <https://github.com/getify/Functional-Light-JS>
