---
layout: post
title: "An introduction to Lambda Calculus for JavaScript developers"
image: ""
published: false
---

I have recently become very interested in functional programming, and one of the areas of study that is often cited as significant for functional programmers is lambda calculus.

In this article I want to look at what lambda calculus is, why you might want to learn about it, and explain the key concepts of lambda calculus using both lambda syntax and 'equivalent' JavaScript code snippets.

# Why should I learn lambda calculus?

There are a few reasons to learn lambda calculus, the main ones I can think of are:

1. It embodies some of the most important concepts of functional programming - pure functions, unary functions, currying.
2. The internals of many functional programming languages such as Haskell, are heavily based on lambda calculus - understanding lambda calculus will help you to learn these languages.
3. Many general purpose programming languages, including JavaScript, borrow concepts from lambda calculus, eg. lambda functions, lazy evaluation.
4. It is an interesting area of mathematics that is relatively accessible to those with a minimal maths background.

# What is lambda calculus?

Lambda calculus was invented by Alonzo Church in the 1930s, and is what is known as a 'computational model'. By that, I mean that it is a system which can be used to encode and compute any algorithmic problem.

The computational model most of us are familiar with is the Turing machine. While lambda calculus is rather different to the Turing machine in its approach to computation, the two are formally equivalent - ie. any problem that can by solved using a Turing machine can be solved using lambda calculus, and vice versa.

While lambda calculus can be used to compute pretty much anything, it is very minimalistic in its axioms/rules. However, it is surrounded by quite a lot terminology which can be daunting at first. We will see in this article that much of this terminology is in fact analogous to concepts that most developers will already be familiar with.

## Lambda terms

Lambda calculus is made up of just 3 basic components or lambda terms:

1. **Variable** - a named token used in a function, which will be replaced by concrete arguments when the function is applied.

2. **Abstraction** - a function, made up of a head and a body:

    * The head is a lambda followed by a variable name. 
    * The body is an expression.

3. **Application** - calling a abstraction (function) with a concrete argument.

An **expression** is a variable, an abstraction, or any combination of variables and abstractions.

In addition, parentheses are used in lambda calculus to indicate the order of evaluation.

## A simple example - the identity function

The following function is known as the identity function, it takes a value and returns that same value. In JavaScript we can easily write and invoke this function:

```js
const identity = x => x;

identity(3) // 3
```

We can write the same function using lambda calculus as follows:

$$ \lambda x . x $$

In the above:
  * $$ x $$ is a variable (and also an expression).
  * $$ \lambda x . x $$ is an abstraction (and also an expression).

    * $$ \lambda x $$ is the head of the abstraction
    * $$ x $$ is the body of the abstraction (and also an expression).

To perform an application on this function with lambda calculus, we use the following syntax:

$$ (\lambda x . x) \ 3 $$

$$ [x := 3] $$

$$ 3 $$

$$ [x := 3] $$ indicates that we are going to substituate all instances of $$ x $$ in the function with $$ 3 $$. The above process is known as **beta reduction**, which I will talk about now.

## Beta reduction

In JavaScript when we invoke a (pure) function with its arguments, the function gets evaluated and it returns the evaluated result:

```js
const f = (x, y) => x * y;

f(2, 3); // 6
```

In lambda calculus, the process of applying conrete arguments to an abstraction, and reducing the resulting expression is known as **beta reduction**. Look at the following function:

$$ \lambda x y . x y $$

We can beta reduce this as follows:

$$ (\lambda x y . x y) \ 2 \ 3 $$

$$ [x := 2] $$

$$ (\lambda y . 2 y) \ 3 $$

$$ [y := 3] $$

$$ 6 $$

**Beta normal form** is the most simplified form of an expression, ie. when it cannot be beta reduced the terms in an expression any further. In the above example $$ 6 $$ is the beta normal form of the application.

## Bound and free variables

// TODO: what are these? Combinators.

The functions which we have looked at so far, have all been made up of bound variables. Lets look at an example which has a free variable:

```js
const z = 5;
// z, in the context of our function, is a free variable.
const f = (x, y) => z + (x * y);

f(2, 3); // 11 === z + 6
```

`z` is closed over by our function and is used to calculate the result of any call to `f`. If we express this same function in lambda calculus, we get:

$$ \lambda x y . z + x y $$

And now applying arguments to this function and beta reducing it:

$$ (\lambda x y . z + (x y)) \ 2 \ 3 $$

$$ [x := 2] $$

$$ (\lambda y . z + (2 y)) \ 3 $$

$$ [y := 3] $$

$$ z + 6 $$

The difference being that in our JavaScript function, we assigned a value to `z`, whereas in the lambda calculus, we just evaluated the function as far as we could without knowing $$ z $$. 

Even though it is not a numerical result, $$ z + 6 $$ is the beta normal form of this application.

## Alpha Equivalence

In JavaScript, renaming a function's parameters has no effect on the meaning of the function.

```js
// The following are equivalent.
const myFunc1 = (x, y) => x * x;
const myFunc2 = (y, x) => y * y;
```

Similarly, in lambda calculus renaming variables within a function also has no effect. We say that abstractions which differ only in their variable names are **alpha equivalent**.

$$ \lambda x y . xx $$ and $$ \lambda y x . yy $$ are alpha equivalent.

## Unary functions and currying

Up until now we have been making a simplification, and viewing lambda functions as functions which take 1 or more arguments and return a single value.

This is not actually quite correct. Every lambda function takes a single argument and returns a single result, and this type of function is know as a **unary function** (a function which takes 2 args is a binary function, a function which takes 3 args is known as a ternary function, etc...). 

$$ \lambda x y z . xyz $$

Is actually shorthand for

$$ \lambda x . (\lambda y .  (\lambda z . xyz)) $$

This is function with a single parameter $$ x $$, which when invoked returns another function with a single parameter $$ y $$, which when invoked returns another function with a single parameter $$ z $$, which when invoked returns the value of $ xyz $ 

In javaScript, this functions looks like:

```js
const f = x => y => z => x * y * z;
```

We can invoke this function either by passing all its arguments in a single expression and obtaining a result, or by creating a more specific function by passing only some arguments, and then calling this function later:

```js
// Call our function with 3 arguments.
f(2)(3)(4); // 24

// Call our function with 2 arguments to create a more specialised function.
const multiplyBySix = f(2)(3);

multiplyBySix(4); // 24
```

The process of breaking down a function into a series of unary functions is known as currying. All functions in lambda calculus are curried unary functions, despite that fact that we often use the aforementioned shorthand which makes them appear as if they take multiple arguments.


# Divergence

Just as not all JavaScript function calls can be evaluated to produce a value, not all lambda functions reduce to a beta normal form upon application. Consider the following, somewhat contrived piece of JavaScript:

```js
const f = x => x(x);
f(f);
```

This is a little confusing. We define a function `f`, which is a function which takes a function `x` and calls it with itself as an argument. 

We then call `f` with itself, which will recursively continue to call f with itself, and mayhem ensues. If you execute the above code, you will get a max call stack size exceeded error. The expression cannot be evaluated, and conceptually we can think of what happens as something like:

```js
f(f(f(f(f(f(...)))))); // and so on, until we reach the maximum call stack size.
```

In lambda calculus, we say that a lambda function which cannot be reduced to beta normal form **diverges**. Here is the equivalent of the above expressed with lambda calculus:

$$ (\lambda x . xx) (\lambda x . x x) $$

If we try and beta reduce this we get stuck in an infinite loop.

$$ [x := \lambda x . x x] $$

$$ (\lambda x . xx) (\lambda x . x x) $$

# Associativity and precendence

In my mind this is the hardest part of lambda calculus, it is basically, deciding where to place brackets and hence give an order to the evaluation of a lambda function.

There are 3 rules to remember:

* **Application has higher precendence than abstraction.**

  We group applications before we group abstractions.

  $$ \lambda xy . x y $$ 

  $$ \lambda xy . (x y) $$

  In the above, the application of $$ x $$ to $$ y $$ is grouped. This is in contrast to if we tried to group abstractions first, which would be **the incorrect way**:

  $$ (\lambda xy . x) y $$

* **Application is left associative, meaning that we group terms to the left.**

  $$ \lambda wxyz . w x y z $$
  
  $$ \lambda wxyz . (((w x) y) z) $$

* **Abstraction is right associative meaning than we grouped terms to the right.**

  $$ \lambda x.x \lambda y.y \lambda z.z $$

  $$ \lambda x.(x ( \lambda y.(y ( \lambda z.z)))) $$

# Combinators

TODO: remove?

Combinators are expressions which have no free variables - they simply combine the arguments that they are given.

$$ \lambda y z . y z $$ is a combinator as all variables in the body of the abstraction are bound to a variable in the head

$$ \lambda y . z $$ is not a combinator, as $$ z $$ is a free variable not bound to anything in the head of the abstraction.

## Conclusion

TODO: next article combinators.
