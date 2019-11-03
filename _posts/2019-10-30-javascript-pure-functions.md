---
layout: post
title: "An introduction to pure functions in JavaScript"
image: "/images/pure-functions-in-js/cupcake-machine.jpg"
published: true
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

    A pure function must not depend on the value of any variable external to the function. Note that it is acceptable for a pure function to depend on a constant external to the function.

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
    function reverseAndJoin(wordsArray) {
        // Use a mutating method on the array passed in.
        const reversed = wordsArray.reverse();
        const reversedAndJoined = reversed.join(', ');
        // Modify some global state
        window.reversedAndJoined = reversedAndJoined;
    }
    ```

    Now think about when we call this function:

    ```js
    let wordsArray = ['One', 'Two', 'Three'];
    reverseAndJoin(wordsArray);
    ``` 

    We have no idea what is going on just by looking at the call site. It is not obvious how we obtain the result of the work done by this function, or that `wordsArray` has been reversed. 
    
    Okay, an extreme example - but I expect you have worked on a project where code not too dissimilar to this has caused issues. It could be refactored to something like the following:

    ```js
    function reverseAndJoin(wordsArray) {
        const reversed = [ ...wordsArray].reverse();
        return reversed.join(', ');
    }
    ```

    Now the call site of the function will look like this:

    ```js
    let wordsArray = ['One', 'Two', 'Three'];
    let reversedAndJoined = reverseAndJoin(wordsArray);
    ```

    This function is very easy to reason about without having to dive into the implementation - the work done by the function is now given back in its return value, and `wordsArray` is not unexpectedly modified.

    The caveat to this point is that at the call site of a function we do not know if it is pure or not, and for pure functions to make our code easier to reason about, we need to know this. In JavaScript, the best we can do is have some kind of methodology or framework whereby particular functions within an application are kept pure by convention (such as redux).

2. **They are easy to unit test.**

    A pure function is easier to test because we don't have any sources of side effects or side causes. This means no external dependencies to mock, and no need to test that some particular side effect worked as expected.

    Using `jasmine`, a unit test for a pure function becomes very simple, and in fact can often be expressed a one liner:

    ```js
    function add(a, b) { 
        return a + b; 
    }

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

Note that although we categorise these functions as pure or impure, it is not always clear cut as to whether a function is pure in the technical sense, based on the formal definition that we gave above. In JavaScript it is rather difficult to make a 'watertight' pure function, that will behave as a pure function no matter what the consumer does with it.

## Examples of impure functions

There are a few common ways in which a function can be impure in JavaScript, lets take a look at each of these in turn:

1. **It mutates non-local state directly.**

    ```js
    let resultArray;
    function appendToArray(myArray, item) {
        resultArray = [ ...myArray, item ];
    }
    ```

    This is a pretty simple example of a side effect. The function is impure because it modifies the global window object. 
    
    To make this into a pure function, we can just return the result instead of setting a variable:

    ```js
    function appendToArray(myArray, item) {
        return [ ...myArray, item ];
    }
    ```

2. **It mutates it's arguments.**

    ```js
    function appendToArray(myArray, item) {
        return myArray.push(item);
    }
    ```

    This function may look pure at first glance, but in fact it has side effects. The array `push` method mutates `myArray` - this non-local state mutation is a very common cause of bugs. 

    It is a good idea to be aware of which JavaScript array methods are [Mutator methods](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/prototype#Mutator_methods). If we need to use one of these methods then we can then create a copy of the array and mutate the copy instead:

    ```js
    function appendToArray(myArray, item) {
        return ([ ...myArray ]).push(item);
    }
    ```

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
    
    To make this function pure, we could simply return the result of the function and not modify the `this` binding:


    ```js
    let adder = {
        add: function(a, b) {
            return a + b;
        },
    };
    ```

4. **It's return value depends on non-local state.**

    ```js
    let currentWord = 'hello ';
    function appendToWord(value) {
        return `${currentWord}${value}`
    }
    ```

    This is an example of a function with a side cause - the return value depends on the value of the non-local `currentWord` variable. 
    
    It is worth noting that we don't know for sure that `currentWord` is ever reassigned, but from reading the code and considering the variable name we can take an educated guess that `currentWord` will likely be reassigned at some point during the program's execution - hence we will categorise this function as impure. If `currentWord` were declared using the `const` keyword (or even declared using `let` and treated as constant), then we could consider `appendToWord` to be a pure function.

    In order to make `appendToWord` a pure function, we can give the function a `word` parameter, and then pass `currentWord` as an argument:

    ```js
    let currentWord = 'hello ';
    function appendToWord(word, value) {
        return `${word}${value}`
    }
    ```

    Its usage would be:

    ```js
    const phrase = appendToWord(currentWord, 'world');
    ```

5. **It's return value depends on its `this` binding**

    ```js
    let appender = {
        word: 'hello ',
        appendToWord: function(value) {
            return `${this.word}${value}`;
        },
    };
    ```

    This is another example of a function with a side cause - in this case the return value of the function depends on its `this` binding. 
    
    To make this into a pure function, we could do something similar to the above case, and force the consumer to pass in a `word` argument.
    ```js
    let appender = {
        appendToWord: function(word, value) {
            return `${word}${value}`;
        },
    };
    ```


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

`add` and `square` do not have any side effects or side causes and so are pure functions.

For `addThenSquare`, things are not quite so clear cut. Technically, we could later redefine `add` or `square` to behave differently, and so `addThenSquare` cannot be considered pure in the formal sense. 

In practice, however, it would be unusual, and bad practice to redefine these functions. It is reasonable to assume that `add` and `square` will not be redefined, and so we will categorise `addThenSquare` as a pure function.

# How to use pure functions in your JavaScript application?

Now that you understand what a pure function is, and the benefits of using pure functions, the question is how and when to use them in your code. 

Clearly, not every function you write can or should be pure. Without side effects, we cannot manipulate the DOM, update a database or make a HTTP request - in fact, by definition an application with no side effects has no observable output.

The key point is to organise your application in such a way that particular parts of your application handle side effects, while other parts are made up of pure functions. Side effects should not be sprinkled throughout your codebase, but should be delegated to a particular layer within your application. Patterns such as [redux](https://redux.js.org/introduction/getting-started) can help you with this.

# Conclusion

We have looked at what a pure function is, the benefits of using pure functions in your code, and some examples of pure and impure functions. Finally, we looked an how and when to use pure functions. 

You should now be at a point where you can utilise pure functions in order to write cleaner, more readable, more performant code. An understanding of pure functions is also a great starting point for learning functional programming!

# Helpful Resources

The following were really helpful for writing this article, and I very much recommend them as further reading:

* <https://github.com/getify/Functional-Light-JS>
