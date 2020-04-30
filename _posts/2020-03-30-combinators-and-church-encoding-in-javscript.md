---
layout: post
title: "Lambda calculus explained through JavaScript: combinators and Church encodings"
image: ""
published: false
---

In this article I will talk about Church encoding, the mechanism by which we can represent operators and data using lambda calculus. In particular I will use JavaScript to demonstrate how we can use lambda calculus to implement boolean logic and basic arithmetic.

This is a follow up to my previous article, [An introduction to Lambda Calculus, explained through JavaScript](http://willtaylor.blog/an-introduction-to-lambda-calculus-explained-through-javascript/), which I recommend reading first if you are not familiar with lambda calculus.

Much of what I write about in this article is inspired by the puzzle book [To Mock a Mockingbird](https://www.amazon.co.uk/Mock-Mockingbird-Other-Logic-Puzzles/dp/0192801422/ref=sr_1_1?dchild=1&keywords=to+mock+a+mockingbird&qid=1587458479&sr=8-1), by Raymond Smullyan, and also the 2 part YouTube series, [A Flock of Functions](https://www.youtube.com/watch?v=3VQ382QG-y4&t=2850s) by Gabrial Lebec, both of which I very much enjoyed.

# Church Encoding

With lambda calculus, all we get out of the box are variables, function abstraction, and function application. Essentially we have a language where the only primitive data type is a function!

Amazingly, from this we can implement operators, data structures and formal systems such as boolean logic and arithmetic. In fact, we can compute anything - lambda calculus is Turing complete!

The way we do this is through Church Encoding. Shortly we will look at how we can implement Church encodings for boolean logic and arithmetic of natural numbers.

# Combinators

A explained in my previous article, combinators are simply (pure) functions where all variables in the body of the function are bound to a variable in the head.

There are certain combinators which crop up again and again and have useful applications, and we will introduce some of these now. Some of them may seem trivial and pointless to begin with, but we will see how they can be used to implement formal systems and perform useful computations later on.

You will notice that most of the popular combinators happen to be named after birds, and this is no coincidence! These names originate from a popular puzzle book, [To Mock a Mockingbird](https://www.amazon.co.uk/Mock-Mockingbird-Other-Logic-Puzzles/dp/0192801422/ref=sr_1_1?dchild=1&keywords=to+mock+a+mockingbird&qid=1587458479&sr=8-1), whose author Raymond Smullyan decided to give the combinators bird names, in honour of the mathematician and combinatory logician Haskell Curry`s love of birds and bird watching.

### The Identity (I) combinator

The Identity combinator is the simplest combinator of all, it is a function which takes a single argument and returns that argument.

In JavaScript:

```js
const I = a => a;

I(4); // 4
```

And lambda calculus:

$$ \lambda x . x $$

### The Kestrel (K) Combinator

The Kestrel is a function which takes two arguments and returns the first argument.

In JavaScript:

```js
const K = a => b => a;

K(4)(2); // 4
```

And lambda calculus:

$$ \lambda xy . x $$

We can partially apply this function to give us a constant function, ie. a function which will always return the same value no matter what argument you give it.

```js
const K5 = K(5);

K5('Whatever'); // 5
```

### The Kite (KI) Combinator

The Kite is a function which takes two arguments and returns the second argument.

In JavaScript:

```js
const KI = a => b => b;

KI(4)(2); // 2
```

And lambda calculus:

$$ \lambda xy . y $$

### The Cardinal (KI) Combinator

The Cardinal is a function which takes another function, who has 2 arguments, and flips those arguments.

In JavaScript:

```js
const C = f => a => b => f(b)(a);
```

And lambda calculus:

$$ \lambda fab . fba $$

We can use the Cardinal to flip the arguments of a function as follows:

```js
const subtract = a => b => a - b;
const flipSubtract = C(subtract);
```

### The Mockingbird (M) combinator

The Mockingbird combinator takes a function and applies it to itself.

In JavaScript:

```js
const M = a => a(a);
```

And lambda calculus:

$$ \lambda x . xx $$

This combinator may seem particularly odd at first sight, but it will come in handy, trust me!

## Booleans and boolean logic

Here I will demonstrate how we can use Church encoding to implement boolean logic. The combinators which we have just talked about will help us with this.

We can start by defining our booleans. These will have to be functions as we don't have anything else.

We can define these using the Kestrel and Kite respectively, and building from this we will implement the boolean logic that we are all familiar with.

```js
const True = K;
const False = KI;
```

When we come to test our boolean logic, it will be useful to have a helper function which can convert these back to JavaScript booleans.

```js
Function.prototype.toJsBool = function()  { return this(true)(false); }

True.toJsBool(); // true
False.toJsBool(); // false
```

### Not

So `True` is a function which takes two arguments and always selects the first, while `False` is a function which takes two arguments and always selects the second - we have some very nice symmetry going on here!

With that in mind, you can see that if we were to flip the arguments of the `True` function, we would get the `False` function, and vice versa - this is exactly what we need for a `Not` function.

As we mentioned earlier, the Cardinal is a combinator which takes 2 arguments and flips them, so the Cardinal is our `Not` function!

```js
const Not = C;

Not(True).toJsBool(); // false
Not(False).toJsBool(); // true
```

### If Then Else

For `IfThenElse` we want a function which takes 3 arguments, the first a boolean, the second and third are values of an arbitrary type. If the first argument is `True`, then `IfThenElse` it should return the second argument, otherwise it should return the thrid argument.

As `True` is a function which selects the first of 2 arguments, and `False` is a function which returns the second of 2 arguments, all we need is to pass the second and third arguments to our first (boolean) argument. 

We need the result of calling `IfThenElse` with only 1 of its arguments to simply return that argument. This is the Identity function!

```js
const IfThenElse = I;
console.log(IfThenElse(True)('It was true')('It was false')); // It was true
console.log(IfThenElse(False)('It was true')('It was false')); // It was false
```

This works because we can always substitute a value with the result of calling the identity function with that value.

```js
I(True) === True; // true
```

### Or

For `Or` we need a function which takes 2 boolean values and returns `True` if at least one of them is `True`.

We can use the Mockingbird combinator for this.

```js
const Or = M;
console.log(Or(True)(True).toJsBool()); // true
console.log(Or(False)(True).toJsBool()); // true
console.log(Or(True)(False).toJsBool()); // true
console.log(Or(False)(False).toJsBool()); // false
```

If you can't quite unpick how this works in your head, remember that the Mockingbird takes a function and calls that function with itself, so:

```js
Or(True) === True(True);
Or(False) === False(False);
```

Hence:

```js
Or(True)(True) === True(True)(True);
Or(False)(True) === False(False)(True);
```

### And

For `And` we need a function which takes 2 boolean values and returns `True` if both of those values is `True`.

```js
const And = p => q => p(q)(p);
```
This function basically reads as:

* If the first value is `True`, return the second value.
* If the first value is `False`, return the first value (ie. short circuit to `False`).

```js
console.log(And(True)(True).toJsBool()); // true
console.log(And(False)(True).toJsBool()); // false
console.log(And(True)(False).toJsBool()); // false
console.log(And(False)(False).toJsBool()); // false
```

### Equals

For `Equals` we need a function which takes 2 boolean values and returns `True` if both of those values are the same.

```js
const Equals = p => q => p(q)(Not(q));
```

This function basically reads as:

* If the first value is `True`, return the second value.
* If the first value is `False`, return the negation of the second value.

```js
console.log(Equals(True)(True).toJsBool()); // true
console.log(Equals(False)(True).toJsBool()); // false
console.log(Equals(True)(False).toJsBool()); // false
console.log(Equals(False)(False).toJsBool()); // true
```

## Numbers and arithmetic

TODO: Introduce necessary combinators + explain

### Numbers and Succ

### Addition

### Multiplication

### Division

### Subtraction

## Performance

TODO: explain that it is a stupid way to perform calculation.

## Conclusion

TODO: talk about function composition in JavaScript

## Useful Resources

A flock of functions:

https://www.youtube.com/watch?v=3VQ382QG-y4&t=2850s
https://www.youtube.com/watch?v=pAnLQ9jwN-E

StackBlitz:

https://stackblitz.com/edit/church-encodings


https://blog.ploeh.dk/2018/05/22/church-encoding/The plays sound interesting, might check them out.