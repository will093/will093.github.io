---
layout: post
title: "JavaScript map, filter and reduce functions explained, with examples"
image: "/images/map-filter-reduce/world-map.jpg"
published: false
---

In this article I will explain, with examples, when and why you should use `map`, `filter` and `reduce` in your JavaScript code. I will show you how these methods can make your code more declarative and readable, and also act as a gateway for getting into that functional programming mindset.

First, we will work through a simple example application, written with `for` and `for ... of` loops. We will then rewrite this example using `filter` and `map`, and then rewrite it again using `reduce`. I will discuss the readability and performance of these solutions, and also mention some of the cases where you may actually be better off using `for ... of`. 

Finally, we will take a look at some of the use cases for the most complex of these 3 array methods,`reduce`.

If after reading this article you would like to dive further into learning about JavaScript arrays, I recommend enroling on the Udemy course (affiliate link), [Mastering JavaScript Arrays](https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.2497482&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fmastering-javascript-arrays%2F).

# An example application: Albert Einstein quotes

We are going to build a very simple example application. It is a web page which fetches some famous quotes from [an API endpoint](https://quote-garden.herokuapp.com/quotes/all), processes this data to extract the quotes of Albert Einstein, and then renders these in the DOM.

The full example is available [on StackBlitz](https://stackblitz.com/edit/es6-filtermapreduce).

For the purpose of this section, we will be looking at one particular function from this StackBlitz example, `getEinsteinQuotes`. The purpose of this function is to take an array of quote objects, which will be structured like this:

```js
[
  {
    quoteText: 'I never think of the future. It comes soon enough.',
    quoteAuthor: 'Albert Einstein',
  },
  {
    quoteText: 'In the middle of every difficulty lies opportunity.',
    quoteAuthor: 'Albert Einstein',
  },
  {
    quoteText: 'Courage is going from failure to failure without losing enthusiasm.',
    quoteAuthor: 'Winston Churchill'
  }
]
```

And return an array of just the `quoteText` of each Einstein quote, like this:

```js
[
  'I never think of the future. It comes soon enough.',
  'In the middle of every difficulty lies opportunity.',
]
```

When iterating over arrays, many developers immediately reach for `for`, `forEach` or `for ... of`. These constructs have their uses, but often result code which is imperative, verbose and difficult to read and reason about. 

Here, we will first look at how to implement this `getEinsteinQuotes` function using `for`, and then `for ... of`, before continuing on to see how we can implement the same function using `map`, `filter` and `reduce`.

## First attempt: for loop

First we will see how we can solve this problem using a `for` loop:

```js
/**
 * Loop over the data with a for loop, use an if statement in 
 * order to push only the items we want to an array.
 */
function getEinsteinQuotes(quotes) {
  const einsteinQuotes = [];

  for (let i = 0; i < quotes.length; i++) {
    const quote = quotes[i];
    if (quote.quoteAuthor.includes('Einstein')) {
      einsteinQuotes.push(quote.quoteText);
    }
  }

  return einsteinQuotes;
}
```

This code certainly works, but it has its problems. 

It is highly unreadable, and contains an unnecessary amount of code, both of which will lead to more mistakes and hence more bugs both during implementation and maintenance. In particular, much of the logic is concerned with low level details of working with arrays:

* `let i = 0; i < quotes.length; i++` is specifying how to handle the control flow of our for loop, ie. how to loop over the array.
*  `const einsteinQuotes = [];`, and later `einsteinQuotes.push(quote.quoteText);` is a solution for how to move items from one array into another.

Neither of these things are clearly related to the purpose of our function, our 'business logic'. For a reader of the code they serve as 'fuzz' or obfuscation to quickly understanding what the function does.

## Second attempt: for ... of loop

Now let's see how we can solve the same problem using a [for ... of](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of) loop.

```js
/**
 * Loop over the data with a for ... of loop, use an if statement
 * in order to push only the items we want to an array.
 */
function getEinsteinQuotesList(quotes) {
  const einsteinQuotes = [];

  for (let quote of quotes) {
    if (quote.quoteAuthor.includes('Einstein')) {
      einsteinQuotes.push(quote.quoteText);
    }
  }

  return einsteinQuotes;
}

```

This is slightly better than the `for` loop method, but still not perfect. 

The `for ... of` loop iterates over each item in the array in order. Unlike with the `for` loop, we do not have to manually write any logic for the control flow of the loop.

## Third attempt: filter and map

Now for what I consider to be the best solution, using [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) and [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map):

```js
/**
 * Transform the quotes data using filter and map.
 */
function getEinsteinQuotesList(quotes) {
  return quotes
    .filter(q => q.quoteAuthor.includes("Einstein"))
    .map(q => q.quoteText);
}
```

Here we take a very different approach to either of the first 2 attempts. 

* `filter` takes the quotes array and executes a predicate function (a function which returns `true` or `false`) for each item in the array. The function acts as a test to calculate which items should be filtered out. `filter` then returns a new array containing all items for which the predicate function returns `true`.

* `map` takes the filtered quotes array and executes a callback function for each item in the array. The callback function's job is simply to take a single array item and return a new value (in this case the `quoteText`). `map` returns a new array containing all of these mapped values.

Each of these functions returns an array, which is why they can be chained as they are above. Also, note how `filter` and `map` both take a function as a parameter - we call functions like this higher order functions. 

The advantages of this approach are mostly with readability and conciseness:  

* We have no need to explicitely create an empty array and append to it, our chained filter and map functions return an array of results and so so there is no longer a need for this.
* `filter` and `map` are very readable and declarative - the name of the method instantly tells us a lot the operation we are performing on the data (unlike the `for ... of` loop), and we do not need to manually write any control flow logic for iterating over the array, (unlike the `for` loop).

This type of solution is particularly favoured by those who are familiar with functional programming. This is because it relies on manipulating the data using [pure functions](https://willtaylor.blog/javascript-pure-functions/), and does not mutate any state. Learning to use these JavaScript array methods can act as a gateway or introduction to learning functional programming.

## Bonus attempt: reduce

Finally, lets look at how we can implement the same function using [reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce):

```js
/**
 * Transform the quotes data using reduce.
 */
function getEinsteinQuotesList(quotes) {
  return quotes.reduce(
    (accumulator, q) => q.quoteAuthor.includes("Einstein") ? [ ...accumulator, q.quoteText ]: accumulator, 
  []);
}
```

So this one takes a little longer to get your head around, hence I would generally recommend using `filter` and `map` as the optimum solution (for all but performance). 

However, it is worth taking a look at, and noting that any operation which can be performed using a combination of `map` and `filter` can also be performed using a single `reduce` function.

The idea is that `reduce` iterates over an array, passing accumulated state from one iteration to the next - the final result is the return value of the final iteration. 

In the above, the second argument is an empty array - this is the value passed as the `accumulator` parameter for the first iteration. Each iteration then returns either just returns `accumulator` if the quote is not an Einstein quote, or a new array containing the contents of `accumulator` and the `quoteText` of the current quote if it is an Einstein quote.

This is the hardest of these 3 functions to get your head around, but it has many interesting uses which I will talk about later in this article.

# Performance

The `for` loop and `for ... of` loop (and `forEach` function) are [equivalent performance wise](https://www.youtube.com/watch?v=m9cTaYI95Zc&feature=youtu.be&t=951) in modern JS engines. 

`map`, `filter` and `reduce` are [generally a bit slower](https://dev.to/henryjw/array-map-much-slower-than-for-loop-57if) than the `for` and `for ... of` loops, especially when working with large data sets. 

In our example, the main thing I would like to point out is how in our third attempt, we chained `map` and `filter`, resulting in the function iterating over each array element twice, and creating a new array twice (`map` returns a new array, and so does `filter`). 

It is worth thinking about the fact that when you chain array methods like this, the array will be iterated over and a new array created for each method in the chain.

All in all attempt 3 is almost definitely the least performant, and yet generally it is still probably the best approach for most situations, especially if you are working with a small or medium sized data set. As a general rule, unless you strongly suspect or can show that the code you are working on is a performance bottleneck, it is better to write readable and maintainable code and refactor later on.

# Should I ever use `for` or `for ... of`?

Yes, `for ... of` has many good uses (I would personally always use `for ... of` over `for`).

Firstly, as discussed above `for ... of` can be more performant in certain cases.

Secondly, `for ... of` can and should be used for executing side effects, for example, a call to the DOM API. In fact, going back to our [Einstein quotes example](https://stackblitz.com/edit/es6-filtermapreduce), lets take a look at the `appendQuotesToDOM` function:

```js
function appendQuotesToDOM(quotes) {
  const ul = document.getElementById("quotes");
  // Use a fragment to minise no of DOM reflows (check this)
  const fragment = document.createDocumentFragment();

  // Foreach is appropriate here, as this is a side effect.
  for (let quote of quotes) {
    const el = document.createElement("li");
    el.appendChild(document.createTextNode(quote));
    fragment.appendChild(el);
  };

  ul.appendChild(fragment);
}
```

Here we use a `for ... of` loop in order to append each list item to the document fragment. In this case we are not transforming a data set, we are performing the side effect of appending to a DOM fragment - to use `map` for this would be possible but not advisable because:

1. By convention, the callback provided to `map` should be a [pure function](https://willtaylor.blog/javascript-pure-functions/) which 'projects' or 'maps' one value to another.
2. The callback passed to `map` returns a value each time it is called, and `map` will return an array of those values - but here, we would not have any need for this array of returned values.

All in all, if we were to use `map` here, it would make our code less readable/reasonable, as we would be bending it to do something that most developers would not expect `map` to be used for.

# Further examples of reduce

While in our above example `reduce` did not turn out to be the best solution, this does really not do it justice. `reduce` is perhaps the most powerful of these 3 array methods and it has many interesting use cases. Lets look at a few:

## Summing an array

Finding the sum of an array of numbers is one of the best examples to help you understand how `reduce` works.

```js
const sumArray = (arr) => arr.reduce((acc, n) => return acc + n);

const sum = sumArray([1,2,3,4,5]);
console.log(sum); // 15
```

Each iteration returns the running total. We could have provided `0` as the second argument to `reduce` and it would have been used as an initial value, but we did not. 

Because we did not, the first item in the array is skipped and instead gets passed to the `acc` parameter for the first iteration. ie. for the first iteration `acc` is `1`, `n` is `2`.

## Converting an array to a dictionary

This is an example of `reduce` is a function which I have used in real projects quite a few times in the past. It converts an array of objects to a dictionary (ie. actually just a plain JS object) keyed by a specified property:

```js
const createDictionary = (arr, key) => {
  return arr.reduce((dict, obj) => {
    dict[obj[key]] = obj;
    return dict;
  }, {});
}

const myArray = [
  { id: '174d0a85', name: 'Jimmy' },
  { id: '9b7e77e2', name: 'Sarah' },
]

const myDict = createDictionary(myArray, 'id');
console.log(myDict);
// {
//   '174d0a85': { id: '174d0a85', name: 'Jimmy' },
//   '9b7e77e2': { id: '9b7e77e2', name: 'Sarah' },
// }
```
## Implementing pipe and compose functions

This is a great example of `reduce` for those interested in functional programming. 

In functional programming, we are often concerned with composing functions. For this, `pipe` and `compose` functions are often used. 

If you are not familiar with functions like `pipe` (or `compose`) don't worry, the purpose is essentially to pass any number of functions to `pipe` (or `compose`), and have it return a function which, when called, will pass the return value of each of these functions from one to the next, with `pipe` finally returning the return value of the final function passed to it. 

The only difference between `pipe` and `compose` is that `pipe` calls the functions passed to it from left to right, while `compose` calls the functions passed to it from right to left.

`pipe` can be implemented using `reduce`, and then used, as follows:

```js
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);

// The functions which we will compose using pipe.
const subtract1 = x => x - 1;
const square = x => x * x;
const add1 = x => x + 1;

const subtract1ThenSquareThenAdd1 = pipe(subtract1, square, add1);
const result = subtract1ThenSquareThenAdd1(6);
console.log(result); // 26
```

Here, it helps to think of our reduce as building as passing the results of the return value of one function call to the next in a nested fashion. ie. we can conceptualise our piped function as being (a function) equivalent to the following:

```js
const subtract1ThenSquareThenAdd1 = x => add1(square(subtract1(x)));
```

`compose` can be implemented using a very similar JavaScript array method, [reduceRight](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight). This method is exactly the same as `reduce`, except that it iterates through the array backwards:

```js
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);

// The functions which we will compose.
const subtract1 = x => x - 1;
const square = x => x * x;
const add1 = x => x + 1;

const add1ThenSquareThenSubtract1 = compose(subtract1, square, add1)
const result = add1ThenSquareThenSubtract1(6);
console.log(result); // 48
```

`compose` does exactly the same as `pipe`, but calls the functions pass to it from right to left. We can conceptualise our composed function as being (a function) equivalent to the following:

```js
const subtract1ThenSquareThenAdd1 = x => subtract1(square(add1(x)));
```

## Sequentially resolving promises

`reduce` can be used to chain promises so that they execute sequentially. Lets look at a simple example where we fetch data from a number of API endpoints:

```js
const urls = [
  'https://some-api.com/endpoint1',
  'https://some-api.com/endpoint2',
  'https://some-api.com/endpoint3',
];

urls.reduce((previousPromise, url) => {
  return previousPromise.then(() => {
    return fetch(url);
  });
}, Promise.resolve());
```

Each of these requests will be made sequentially - the next one being made when the previous one has received a response. 

If we unwrap this mentally, we can think it as essentially building a promise chain using `then`. And if you are familiar with promises, of course each `then` is executed only once the previous one has resolved. `reduce` creates a chain of promises equivalent to this:

```js
fetch(() => urls[0]).then(() => fetch(urls[1])).then(() => fetch(urls[2]));
```

Alternatively, using modern `async` and `await` syntax:

```js
const urls = [
  'https://some-api.com/endpoint1',
  'https://some-api.com/endpoint2',
  'https://some-api.com/endpoint3',
];

urls.reduce(async (previousPromise, url) => {
  await previousPromise;
  return fetch(url);
}, Promise.resolve());
```

Which we can think of as equivalent to:

```js
await fetch(urls[0]);
await fetch(urls[1]);
await fetch(urls[2]);
```

# Conclusion

In this article we looked at some examples of how and when to use `map`, `filter` and `reduce`, and hopefully this has given you some further insight into how these functions work. 

Now that you understand these 3 methods, there are many other useful array methods you can learn such as [some](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/some), [every](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every), [`includes`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes) and [`find`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find). And for more advanced data manipulation with arrays there are libraries such as [lodash](https://lodash.com/), or for a more functional approach, [ramda](https://ramdajs.com/).

# Useful Resources

If you are looking to learn about JavaScript arrays in more depth, I recommend the following course (affiliate link):

* [Mastering JavaScript Arrays](https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.2497482&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fmastering-javascript-arrays%2F)

In addition, the following were really helpful for writing this article, and I very much recommend them as further reading:

* <https://css-tricks.com/why-using-reduce-to-sequentially-resolve-promises-works/>
* <https://medium.com/free-code-camp/10-ways-to-write-pipe-compose-in-javascript-f6d54c575616>
* <https://thecodebarbarian.com/for-vs-for-each-vs-for-in-vs-for-of-in-javascript.html>