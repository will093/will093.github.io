---
layout: post
title: "An introduction to Lambda Calculus, explained through JavaScript"
image: "/images/introduction-to-lambda-calculus/lambda.jpg"
published: false
---

I have recently become very interested in functional programming - using pure functional languages such as Haskell, as well as functional programming in JavaScript. One of the main areas of study that is often cited as significant for functional programmers is lambda calculus.

In this article I want to look at what lambda calculus is, why you might want to learn about it, and explain the key concepts and the terminology of lambda calculus using both lambda syntax and 'equivalent' JavaScript code snippets.

If after reading this you are interested in learning more about functional programming in JavaScript, I recommend this Udemy course (affiliate link):

[Functional Programming For Beginners With JavaScript](https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.1537962&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Ffunctional-programming-for-beginners-with-javascript%2F)

# Why should I learn lambda calculus?

There are a few reasons to learn lambda calculus, the main ones I can think of are:

1. **It embodies some of the most important concepts of functional programming** 

    For example, pure functions, unary functions, currying.

2. **The internals of many functional programming languages such as Haskell, are heavily based on lambda calculus** 

    Understanding lambda calculus will help you to understand these languages.

3. **Many concepts from lambda calculus are applicable to general purpose languages such as JavaScript.**

    Concepts such as pure functions, unary functions and currying are often used in functional JavaScript.

4. **It is interesting**

    Lambda calculus is an interesting area of mathematics, and is relatively accessible to those with a minimal maths background.

# What is lambda calculus?

Lambda calculus was invented by the mathematician [Alonzo Church](https://en.wikipedia.org/wiki/Alonzo_Church) in the 1930s, and is what is known as a 'computational model'. By that, I mean that it is a system which can be used to encode and compute algorithmic problems.

The computational model most of us are familiar with is the Turing machine. While lambda calculus is rather different to the Turing machine in its approach to computation, the two are formally equivalent - ie. any problem that can by solved using a Turing machine can be solved using lambda calculus, and vice versa.

That is to say, lambda calculus can be used to solve any problem that can be computed using a Turing machine (so anything that anyone has ever programmed)!
## Lambda calculus basics

Lambda calculus is very minimalistic in its rules/axioms. It is made up of just 3 basic components, or **lambda terms**:

1. **Variable** - a named token used in a function, which will be replaced by concrete arguments when the function is applied.

2. **Abstraction** - a function, made up of a head and a body separated by a '$$ . $$'

    * The head is a lambda followed by a variable name. 
    * The body is an expression.

3. **Application** - a function invokation.

An **expression** is a variable, an abstraction, an application, or any combination of these.

In addition, parentheses are used in lambda calculus to indicate the order of evaluation.

This is all we get in lambda calculus. It is quite amazing that using these 3 lambda terms we calculate pretty much anything. 

It is worth noting the absence of any of the constructs of basic arithmetic, such as numbers, addition, subtraction, multiplication, division etc - these are not within the basic axioms of lambda calculus. However, these things can be defined in terms of lambda calculus via [Church Encoding](https://en.wikipedia.org/wiki/Church_encoding), and rather than do this every time we want to use any of these constructs, we can choose to use numbers and basic arithmetic operators for convenience.

## A simple example - the identity function

The following function is known as the identity function, it takes a value and returns that same value. 

In JavaScript we can easily define and execute this function:

```js
const identity = x => x;

identity(3) // 3
```

We can write the same function using lambda calculus as follows:

$$ \lambda x . x $$

In the above:
  * $$ x $$ is a variable.
  * $$ \lambda x . x $$ is an abstraction.

    * $$ \lambda x $$ is the head of the abstraction
    * $$ x $$ is the body of the abstraction.

To perform an application, ie. apply the function to a concrete argument, we use the following syntax:

$$ (\lambda x . x) \ 3 $$

$$ [x := 3] $$

$$ 3 $$

In the above, the $$ [x := 3] $$ syntax indicates that we are going to substitute all instances of $$ x $$ in the function with $$ 3 $$. The above process is known as **beta reduction**, which I will talk more about now.

## Beta reduction

In JavaScript when we invoke a (pure) function with its arguments, the function gets evaluated and it returns the evaluated result:

```js
const f = (x, y) => x(y);

f(z => z + 1, 3); // 4
```

Here, `f` takes 2 arguments - a function `x` and a value `y`, and calls `x` with `y` as an argument.

In lambda calculus, the process of applying concrete arguments to an abstraction, and reducing the resulting expression is known as **beta reduction**. 

The equivalent to the above JavaScript function `f`, when written with lambda calculus, is:

$$ \lambda x y . x y $$

This time, we have an application in the body of the abstraction: $$ x y $$, this means 'call $$ x $$ with $$ y $$ as an argument'. We can pass arguments to the abstraction and beta reduce it as follows:

$$ (\lambda x y . x y) \ (\lambda z . z + 1) \ 3 $$

$$ [x := \lambda z . z + 1] $$

$$ (\lambda y . (\lambda z . z + 1) \ y) \ 3 $$

$$ [z := y] $$

$$ (\lambda y . y + 1) \ 3 $$

$$ [y := 3] $$

$$ 4 $$

**Beta normal form** is the most simplified form of an expression, ie. when it cannot be beta reduced the terms in an expression any further. In the above example $$ 4 $$ is the beta normal form of the application.

## Alpha Equivalence

In JavaScript, renaming a function's parameters has no effect on the behaviour of the function.

```js
// The following are equivalent.
const myFunc1 = (x, y) => x * x;
const myFunc2 = (y, x) => y * y;
```

Similarly, in lambda calculus renaming variables within a function also has no effect. We say that abstractions which differ only in their variable names are **alpha equivalent**.

The following is an example of 2 abstractions which are alpha equivalent:

$$ \lambda x y . xx $$ 

$$ \lambda y x . yy $$ 

## Bound and free variables

In JavaScript, the return value of a function can be calculated us both its parameters, and variables which the function closes over:

```js
const z = 5;
// z, in the context of our function, is a free variable.
const f = (x, y) => z + (x * y);

f(2, 3); // 11 === z + 6
```

Similarly, lambda calculus abstractions can make use of both variables which are defined in the head of the abstraction, and other, 'unknown' variables which are not. In lambda calculus:

* **Bound variables** are variables used in an abstraction which appear in the head of the abstraction.
* **Free variables** are variables used in an abstraction which do not appear in the head, their value is taken to be unknown within the context of the abstraction.

If we express the JavaScript function defined above in lambda calculus, we get:

$$ \lambda x y . z + (x * y) $$

And now applying arguments to this function and beta reducing it:

$$ (\lambda x y . z + (x * y)) \ 2 \ 3 $$

$$ [x := 2] $$

$$ (\lambda y . z + (2 * y)) \ 3 $$

$$ [y := 3] $$

$$ z + 6 $$

The notable difference here is that in our JavaScript function, we assigned a value to `z`, whereas in the lambda calculus, we just evaluated the function as far as we could without knowing $$ z $$. 

Even though it is not a numerical result, $$ z + 6 $$ is the beta normal form of this application.

## Unary functions and currying

Up until now we have been making a simplification, and viewing lambda functions as functions which take one or more arguments and return a single value.

This is *not actually correct*. 

Every lambda function takes a single argument and returns a single result, and this type of function is know as a **unary function**. 

$$ \lambda x y z . x * y * z $$

Is actually shorthand for

$$ \lambda x . (\lambda y .  (\lambda z . x * y * z)) $$

This is function with a single parameter $$ x $$, which when invoked returns another function with a single parameter $$ y $$, which when invoked returns another function with a single parameter $$ z $$, which when invoked returns the value of $$ x * y * z $$ 

In JavaScript, the above function looks like:

```js
const f = x => y => z => x * y * z;
```

If you have done any functional programming in JavaScript then this way of writing a function may look familiar. Taking a function which takes many arguments, and breaking it down into a series of unary functions (which each take a single argument) is known as **currying**. 

We can invoke the above function either by passing all of its arguments and immediately obtaining the final result, or, we can first create a more specific function by passing only some arguments, and then call this function later to get the final result.

```js
// Call our function with 3 arguments.
f(2)(3)(4); // 24

// Call our function with 2 arguments to create a more specialised function.
const multiplyBySix = f(2)(3);

multiplyBySix(4); // 24
```

In JavaScript we have the choice to either use curried unary functions, or functions which take more than one argument. 

In lambda calculus, all functions are curried unary functions, despite that fact that we often use the aforementioned shorthand which makes them appear as if they take multiple arguments.

# Divergence

Just as not all JavaScript function calls can be evaluated to produce a value, not all lambda functions reduce to a beta normal form upon application. Consider the following, somewhat contrived piece of JavaScript:

```js
const f = x => x(x);
f(f);
```

This is a little confusing. We define a function `f`, which takes a function `x` and calls it with itself as an argument. On the next line, we call `f` with itself, which will recursively continue to call f with itself, and mayhem ensues. 

If you execute the above code, you will get a max call stack size exceeded error. Conceptually we can think of what happens as something like:

```js
f(f(f(f(f(f(...)))))); // and so on, until we reach the maximum call stack size.
```

In lambda calculus, we say that a lambda function which cannot be reduced to beta normal form **diverges**. Here is the equivalent of the above expressed with lambda calculus:

$$ (\lambda x . x x) (\lambda x . x x) $$

If we try and beta reduce this we get stuck in an infinite loop:

$$ [x := \lambda x . x x] $$

$$ (\lambda x . x x) (\lambda x . x x) $$

# Associativity and precendence

In JavaScript, we have the concepts of [associativity and precendence](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence) for operators and function calls. 

Associativity and precendence are how the JavaScript interpretter determines the order of evaluation of expressions, or 'where it puts its parentheses'. Consider the following:

```js
const f1 = x => y => z => x / y / z / 2 + 7;
f1(1)(2)(3) // 7.083333333333333
```

Some division and addition has gone on, but it is really not clear in what order, because of the lack of parentheses. Using JavaScript`s rules for precendence and associativity:

* Division (`/`) has higher precendence than addition (`+`), so we divide first.
* Division is left associative, meaning that when there are consecutive division operators, we group operations to the left.

Knowing this, we figure out where 

```js
const f1 = x => y => z => x / y / z / 2 + 7;
const f2 = x => y => z => (x / y / z / 2) + 7; // Division has higher precendence than addition.
const f3 = x => y => z => (((x / y) / z) / 2) + 7; // Division is left associative

f1(1)(2)(3) // 7.083333333333333
f2(1)(2)(3) // 7.083333333333333
f3(1)(2)(3) // 7.083333333333333
```
Now lets take a look and precendence and associativity in lambda calculus. This is the part of lambda calculus that I most struggled to get my head around. 

There are 3 rules to remember:

* **Application has higher precendence than abstraction.**

  ie. We group applications before we group abstractions.

  $$ \lambda xy . x y $$ 

  $$ \lambda xy . (x y) $$

  In the above, we place parentheses in order to group the application of $$ x $$ to $$ y $$. This is in contrast to the result we might get if we tried to group abstractions first, which would be **the incorrect way**:

  $$ (\lambda xy . x) \ y $$

* **Application is left associative, meaning that we group terms to the left.**

  $$ \lambda wxyz . w x y z $$
  
  $$ \lambda wxyz . (((w x) y) z) $$

* **Abstraction is right associative meaning than we grouped terms to the right.**

  $$ \lambda x.x \lambda y.y \lambda z.z $$

  $$ (\lambda x.x ( \lambda y.y ( \lambda z.z))) $$

## Conclusion

We have talked about what lambda calculus is, why you might want to learn it, and discussed the basic concepts of lambda calculus - explained using concepts which should be familiar to JavaScript developers. 

If you found this article interesting please check back here soon for my upcoming article about Church Encoding, where we will see how we can implement numbers and basic arithmetic using only lambda calculus, again explained using JavaScript.

# Helpful Resources

I highly recommend the following Udemy course if you are interested in learning more about functional programming in JavaScript:

<figure>
  <a href="https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.1537962&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Ffunctional-programming-for-beginners-with-javascript%2F">
      <img src="https://i.udemycdn.com/course/480x270/1537962_e309_2.jpg"/>
      <figcaption>Functional Programming For Beginners With JavaScript</figcaption>
  </a>
</figure>

In addition, the following were really helpful for writing this article, and I very much recommend them as further reading:

* <https://www.youtube.com/watch?v=eis11j_iGMs>
* <https://lucasfcosta.com/2018/07/29/An-Introduction-to-Lambda-Calculus-Part-1.html>
* <https://stackoverflow.com/questions/55282692/lambda-calculus-precedence-of-application-and-abstraction>


