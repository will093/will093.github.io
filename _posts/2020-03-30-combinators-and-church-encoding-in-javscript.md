---
layout: post
title: "Lambda calculus explained through JavaScript: combinators and Church encoding"
image: "/images/church-encoding/church.jpg"
published: false
---

In this article I will talk about Church encoding, the mechanism by which we can represent operators and data using lambda calculus. In particular I will use JavaScript to demonstrate how we can use lambda calculus to implement boolean logic and basic arithmetic.

This is a follow up to my previous article, [An introduction to Lambda Calculus, explained through JavaScript](http://willtaylor.blog/an-introduction-to-lambda-calculus-explained-through-javascript/), which I recommend reading first if you are not familiar with lambda calculus.

I have created a [StackBlitz demo](https://stackblitz.com/edit/church-encodings), which contains examples of all of the functions discussed and used in this article.

Much of what I write about in this article is inspired by the puzzle book To Mock a Mockingbird, by Raymond Smullyan, and also the 2 part YouTube series, [A Flock of Functions](https://www.youtube.com/watch?v=3VQ382QG-y4&t=2850s) by Gabrial Lebec.

# What is Church encoding?

With lambda calculus, all we get out of the box are variables, function abstraction, and function application. Essentially we have a language where the only primitive data type is a function!

Amazingly, from this we can implement operators, data structures and formal systems such as boolean logic and arithmetic. In fact, we can compute anything - lambda calculus is Turing complete!

The way we do this is through Church encoding (named after [Alonzo Church](https://en.wikipedia.org/wiki/Alonzo_Church)). Shortly we will look at how we can implement Church encoding for boolean logic and arithmetic of natural numbers.

# Combinators

Combinators are simply (pure) functions where all variables in the body of the function are bound to a variable in the head.

A simple example of this in Lambda calculus:

$$ \lambda x y . x $$

And in JavaScript:

```js
const combinator = (x, y) => x

const z = 3;
const notACombinator = (x, y) => z;
```

There are certain combinators which crop up again and again and have useful applications, and we will introduce some of these next. Some of them may seem trivial and pointless to begin with, but we will see how they can be used to implement formal systems and perform useful computations later on.

You will notice that most of the popular combinators happen to be named after birds, and this is no coincidence! These names originate from a popular puzzle book, [To Mock a Mockingbird](https://www.amazon.co.uk/Mock-Mockingbird-Other-Logic-Puzzles/dp/0192801422/ref=sr_1_1?dchild=1&keywords=to+mock+a+mockingbird&qid=1587458479&sr=8-1), whose author Raymond Smullyan decided to give the combinators bird names, in honour of the mathematician and combinatory logician Haskell Curry`s love of birds and bird watching.

## The Identity (I) combinator

The Identity combinator is the simplest combinator of all, it is a function which takes a single argument and returns that argument.

In lambda calculus:

$$ \lambda x . x $$

In JavaScript:

```js
const I = a => a;

I(4); // 4
```

## The Kestrel (K) Combinator

The Kestrel is a function which takes two arguments and returns the first argument.

In lambda calculus:

$$ \lambda xy . x $$

In JavaScript:

```js
const K = a => b => a;

K(4)(2); // 4
```

We can partially apply this function to give us a constant function, ie. a function which will always return the same value no matter what argument you give it.

```js
const K5 = K(5);

K5('Whatever'); // 5
```

## The Kite (KI) Combinator

The Kite is a function which takes two arguments and returns the second argument.

In lambda calculus:

$$ \lambda xy . y $$

In JavaScript:

```js
const KI = a => b => b;

KI(4)(2); // 2
```

## The Cardinal (KI) Combinator

The Cardinal is a function which takes another function, who has 2 arguments, and flips those arguments.

In lambda calculus:

$$ \lambda fab . fba $$

In JavaScript:

```js
const C = f => a => b => f(b)(a);
```
We can use the Cardinal to flip the arguments of a function as follows:

```js
const subtract = a => b => a - b;
const flipSubtract = C(subtract);
```

## The Mockingbird (M) combinator

The Mockingbird combinator takes a function and applies it to itself.

In lambda calculus:

$$ \lambda x . xx $$

In JavaScript:

```js
const M = a => a(a);
```

This combinator may seem particularly odd at first sight, but it will come in handy, trust me!

# Booleans and boolean logic

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

## Not

So `True` is a function which takes two arguments and always selects the first, while `False` is a function which takes two arguments and always selects the second - we have some very nice symmetry going on here!

With that in mind, you can see that if we were to flip the arguments of the `True` function, we would get the `False` function, and vice versa - this is exactly what we need for a `not` function.

As we mentioned earlier, the Cardinal is a combinator which takes 2 arguments and flips them, so the Cardinal is our `not` function!

```js
const not = C;

not(True).toJsBool(); // false
not(False).toJsBool(); // true
```

## If Then Else

For `ifThenElse` we want a function which takes 3 arguments, the first a boolean, the second and third are values of an arbitrary type. If the first argument is `True`, then `ifThenElse` it should return the second argument, otherwise it should return the thrid argument.

As `True` is a function which selects the first of 2 arguments, and `False` is a function which returns the second of 2 arguments, all we need is to pass the second and third arguments to our first (boolean) argument. 

It follows that we need the result of calling `ifThenElse` with only 1 of its (boolean) arguments to simply return that argument. The function that does this is the Identity function!

```js
const ifThenElse = I;
console.log(ifThenElse(True)('It was true')('It was false')); // It was true
console.log(ifThenElse(False)('It was true')('It was false')); // It was false
```

This works because we can always substitute a value with the result of calling the identity function with that value.

```js
I(True) === True; // true
```

## Or

For `or` we need a function which takes 2 boolean values and returns `True` if at least one of them is `True`.

We can use the Mockingbird combinator for this.

```js
const or = M;
console.log(or(True)(True).toJsBool()); // true
console.log(or(False)(True).toJsBool()); // true
console.log(or(True)(False).toJsBool()); // true
console.log(or(False)(False).toJsBool()); // false
```

If you can't quite unpick how this works in your head, remember that the Mockingbird takes a function and calls that function with itself, so:

```js
or(True) === True(True);
or(False) === False(False);
```

Hence:

```js
or(True)(True) === True(True)(True);
or(False)(True) === False(False)(True);
```

## And

For `and` we need a function which takes 2 boolean values and returns `True` if both of those values is `True`.

```js
const and = p => q => p(q)(p);
```
This function basically reads as:

* If the first value is `True`, return the second value.
* If the first value is `False`, return the first value (ie. short circuit to `False`).

```js
console.log(and(True)(True).toJsBool()); // true
console.log(and(False)(True).toJsBool()); // false
console.log(and(True)(False).toJsBool()); // false
console.log(and(False)(False).toJsBool()); // false
```

## Equals

For `equals` we need a function which takes 2 boolean values and returns `True` if both of those values are the same.

```js
const equals = p => q => p(q)(not(q));
```

This function basically reads as:

* If the first value is `True`, return the second value.
* If the first value is `False`, return the negation of the second value.

```js
console.log(equals(True)(True).toJsBool()); // true
console.log(equals(False)(True).toJsBool()); // false
console.log(equals(True)(False).toJsBool()); // false
console.log(equals(False)(False).toJsBool()); // true
```

# More combinators

That is as far as we will go with boolean logic! Soon we will move on to implementing numbers and arithmetic - but first I will introduce a few more combinators which will help us with this.

## Bluebird combinator

The bluebird combinator takes three arguments, and calls the first argument with the result of calling the second argument with the third. 

In lambda calculus:

$$ \lambda f g a . f(ga) $$

In JavaScript:

```js
const B = f => g => a => f(g(a));
```

Sounds slightly confusing described in this way, but this combinator, when called with just two of its arguments, actually just composes two unary functions (a unary function is a function which takes a single argument). If you are not familiar with function composition, this means chaining functions in such a way that the return value of one is passed to the next:

```js
const add3 = x => x + 3;
const double = x => x * 2;

const doubleThenAdd3 = B(add3)(double);
doubleThenAdd3(5); // 13
```

Notice that in this case the composition goes from right to left - the argument is doubled and then 3 is added. You can visualise how the composed function works by defining the function like this, which is its less elegant equivalent:

```js
const doubleThenAdd3 = x => add3(double(x));
doubleThenAdd3(5); // 13
```

## Thrush combinator

The thrush combinator takes two arguments, and calls the second with the first.

In lambda calculus:

$$ \lambda a f . fa $$

In JavaScript:

```js
const TH = a => b => b(a);
```

## Vireo combinator

The vireo combinator takes three arguments, and calls the third with the first and second. 

In lambda calculus:

$$ \lambda a b f . fab $$

In JavaScript:

```js
const V = a => b => f => f(a)(b);
```

This combinator is very interesting, as we can use it in combination with the Kestrel and Kite combinators to create a 2-tuple or pair data structure.

```js
const pair = V;
const fst = a => a(K); // Kestrel is a function which takes 2 arguments and selects the first.
const snd = a => a(KI); // Kite is a function which takes 2 arguments and selects the second.

const myPair = pair(3)(4);
fst(myPair); // 3
snd(myPair); // 4
```

## Blackbird combinator

Remember, we said that Bluebird was a combinator which handled composition of unary functions. Well the blackbird is a combinator that composes a binary function (a function which takes two arguments) with a unary function.

In lambda calculus:

$$ \lambda f g a b . f(gab) $$

In JavaScript:

```js
const BL = f => g => a => b => f(g(a)(b));
```

As a function can only have one return value, the binary function must be the first in the chain, and the unary function must be the second in the chain.

```js
const add = a => b => a + b;
const double = a => a * 2;

const addThenDouble = BL(double)(add);
addThenDouble(4)(7); // 22;
```

It is worth noting that it is much more satisfying (although somewhat hard to get your head around) defining this function in terms of the Bluebird. ie. it is the composition of composition and composition...

```js
const BL = B(B)(B);
```

## Numbers and arithmetic

Now that we have some more combinators in our toolbelt, we will implement natural numbers (zero and positive integers) and arithmetic.

### Numbers and Succ

We can define numbers in terms of number of times a function is applied. For example one is the application of a function with a value once:

```js
const one = f => a => f(a);
```

Subsequent numbers apply a function to an argument n times. Eg.

```js
const two = f => a => f(f(a));
const three = f => a => f(f(f(a)));
const four = f => a => f(f(f(f(a))));
const five = f => a => f(f(f(f(f(a)))));
```

And following on from that idea, zero applies a function to the second zero times:

```js
const zero = f => a => a;
```

How we really want to do this is to start with just zero (which you may notice from the above implementation is equivalent to the Kite combinator), and use a function to generate the subsequent numbers. 

This is our `succ` function, which gets the successor of a number:

```js
const succ = n => f => f(n(f));
```

Or alternatively, implementing this with the Bluebird (composition) combinator:

```js
const succ = n => f => B(f)(n(f));
```

Each of our Church encoded numbers: `one`, `two`, `three` etc.. are functions which apply a function to an argument `n` times. `succ` takes a Church encoded number `n` as an argument, and returns a function which will apply its first argument `f` to its second argument `n + 1` times.

We can implemented positive integers using this `succ` function:

```js
const one = succ(zero);
const two = succ(one);
const three = succ(two);
const four = succ(three);
const five = succ(four);
```

To test our implementation of Church numbers, we will define a `toJsNum` function:

```js
Function.prototype.toJsNum = function() {
  return this(x => x + 1)(0);
};

three.toJsNum(); // 3
five.toJsNum(); // 5
```

### Addition

For `add` we need a function which takes two Church encoded numbers and adds them together.

```js
const add = n => k => n(succ)(k);
```

This essentially says 'give me the nth successor of k'.

```js
add(one)(one).toJsNum(); // 2
add(three)(five).toJsNum(); // 8;
```

### Multiplication

For multiplication we need a function which multiplies 2 church numbers together. 

Our function should take an encoded Church number which applies a function to an argument `n` times, and another encoded Church number, which applies a function to an argument `m` times, and return a function which applies a given function to an argument `n*m` times.

Something like this:

```js
(a => three(five(a))).toJsNum(); // 15
```

But looking at the above we can see this is equivalent to the Bluebird combinator, or composition. Multiplication is simply composition!

```js
const mult = B;

mult(two)(two).toJsNum(); // 4
mult(three)(five).toJsNum(); // 15
```

### Powers/exponentiation

Our `pow` function is the Thrush:

```js
const pow = TH;

pow(three)(four).toJsNum() // 81
```

The above is essentially saying, apply our function three (which itself applies a function 3 times) four times, passing the result of each application into the next. So it results in a function which applies a function 3 * 3 * 3 * 3 (ie. 81) times.

```js
four(three).toJsNum() // 81
```

### Pred function and subtraction

Out of the operations we will look at, this is the most difficult to implement. 

Defining a `pred` function will prove to be more difficult than defining a `succ` function. We will do it by using the Vireo, and the pair data structure which we introduced earlier.

```js
const pair = V;
const fst = a => a(K);
const snd = a => a(KI);
```

Now lets define the following function which takes a pair of church encoded numbers and converts it to a nice readable JS string, which will help us with visualising what we are about to do:

```js
Function.prototype.toJsPair = function() {
  return `(${fst(this).toJsNum()}, ${snd(this).toJsNum()})`;
};

pair(one)(three).toJsPair(); // (1, 3)
```

```js
const phi = p => V(snd(p))(succ(snd(p)));
```

This `phi` function takes a pair, and returns another pair where:

* The first item is equal to the second item in the given argument.
* The second item is equal to the successor of the second item in the given argument.

```js
const myPair = pair(three, five);
phi(myPair).toJsPair(); // (5, 6)
```

So, how can this help us define a predecessor or subtraction function. Well, look at the following:

```js
const pair0 = pair(zero, zero);
const pair1 = phi(pair0);
const pair2 = phi(pair1);
const pair3 = phi(pair2);

phi(pair0).toJsPair(); // (0, 0)
phi(pair1).toJsPair(); // (0, 1)
phi(pair2).toJsPair(); // (1, 2)
phi(pair3).toJsPair(); // (2, 3)
```

By starting with a `pair` of `zero`s, and applying `phi` n times, we find that we get a pair whose first element is equal to `n-1`, the predecessor of `n`! We can write a function for this as follows:

```js
const pred = n => fst(n(phi)(pair(zero)(zero)));

pred(five).toJsNum(); // 4
```

Note that we have the limitation that this function considers `zero` to be the predecessor of `zero`, and hence there are no negative numbers.

```js
pred(zero).toJsNum(); // 0
```

And `pred` is to subtraction as `succ` is to addition, so we may define our subtraction function as:

```js
const subtract = n => k => k(pred)(n);

subtract(four, three);
```

### Greater than, Less than, greater than or equal to, less than or equal to

We can define our `lessThanOrEqual` by subtracting the second argument from the first, and checking if it is zero.

```js
const lessThanOrEqual = n => k => isZero(subtract(n)(k));

lessThanOrEqual(one)(three).toJsBool(); // true
lessThanOrEqual(three)(one).toJsBool(); // false
lessThanOrEqual(one)(one).toJsBool(); // true
```

For `greaterThanOrEqual` we just need to swap the first and second arguments around. Sounds a lot like the Cardinal!

```js
const greaterThanOrEqual = C(lessThanOrEqual);

greaterThanOrEqual(one)(three).toJsBool(); // false
greaterThanOrEqual(three)(one).toJsBool(); // true
greaterThanOrEqual(one)(one).toJsBool(); // true
```

For `greaterThan`, we can simply compose `not` and `lessThanOrEqual`. Note that as `lessThanOrEqual` is a binary function (takes 2 arguments), we must use the Blackbird for composition, rather than the Bluebird.

```js
const greaterThan = BL(not)(lessThanOrEqual);

greaterThan(one)(three).toJsBool(); // false
greaterThan(three)(one).toJsBool(); // true
greaterThan(one)(one).toJsBool(); // false
```

I will leave it up to you to consider how we can define `lessThan`.

## Conclusion

We have looked at a variety of combinators, and how we can define the formal systems of boolean logic and natural numbers using Church encoding. This is a large topic, and you may have noticed that I left out certain combinators and operations, for example division, the Y-combinator. I decided to leave these outside of the scope of the article, but may write about them in future.

As a final note, while Church encoding is interesting way to show how we can build formal systems using lambda calculus, it is worth noting that it is not efficient for performance, and is not the method used by programming languages such as Haskell which are modelled around lambda calculus.

# Useful Resources

I highly recommend the following Udemy course if you are interested in learning more about functional programming in JavaScript:

<figure>
  <a href="https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.1537962&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Ffunctional-programming-for-beginners-with-javascript%2F">
      <img src="https://i.udemycdn.com/course/480x270/1537962_e309_2.jpg"/>
      <figcaption>Functional Programming For Beginners With JavaScript</figcaption>
  </a>
</figure>

In addition, the following were really helpful for writing this article, and I very much recommend them as further study:

* <https://www.youtube.com/watch?v=3VQ382QG-y4&t=2850s>
* <https://www.youtube.com/watch?v=pAnLQ9jwN-E>
* <https://blog.ploeh.dk/2018/05/22/church-encoding/>