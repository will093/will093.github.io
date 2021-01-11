---
layout: post
title: "Why you should consider building your next project with Svelte"
image: "/images/consider-svelte/legos.jpg"
published: true
---

For the last few months I have been working with Svelte. As a developer who has worked on a lot of Angular codebases as well as the odd React/Vue project, I was pleasantly surprised by how nice Svelte is. 

In this article I want to talk about what Svelte is, and the reasons why you should (or shouldn't) consider it for your next project.

# What is Svelte JS?

One of the things I often heard about Svelte before I first used it was "*It's a compiler, not a framework*". 

This is great, and its true - Svelte is a compiler which takes a bunch of Svelte component files (and JS files) and compiles them into HTML, CSS and JavaScript. However, this sounds very similar to other frameworks such as React which also have a compile step. So what is the difference?

Although other frameworks such as React and Vue have a compile step, this is mostly to transform the JavaScript into something that a browser can understand (and is minified, bundled etc..). The compiled code still contains a lot of 'framework code', for example, code used for diffing and updating the Virtual DOM when anything changes.

In comparision, Svelte does not use a Virtual DOM - it compiles your project to HTML, CSS and JavaScript, where the compiled JS simply updates the DOM directly. Basically, there are no runtime dependencies to be bundled with your source code - what your browser downloads is your project's compiled source code. This is in contrast to React and Vue, where your browser must download your project's compiled source code and also a certain amount of framework code that is needed at runtime.

# What are the advantages of using Svelte over React/Vue/Angular?

In many ways, working with Svelte feels very similar to working with other frontend frameworks. However, there are a few unique features which make it worth considering.

## Reactive statements

Svelte is heavily based on reactivity, but it takes a different approach to React and Vue. 

Reactive statements kind of scared me at first. They can only by used inside of `.svelte` component files, and rely on the dollar syntax (`$:`) which Svelte introduces.

```js
let x = 1;
let y = 2;

setTimeout(() => {
  x = 4;
  y = 5;
}, 1000);

// Reactive statement (variable assignment).
$: z = x + y;

// Reactive statement (side effect).
$: console.log(`x: ${x}, y: ${y}, z: ${z}`); // 'x: 1, y: 2, z: 3' ... 'x: 4, y: 5, z: 9'
```

Reactive statements can used for variable assignment (similar to Vue computed properties) or for arbitrary side effects. Note that the variable assignment form of reactive statement also automatically declares the specified variable.

The idea that these reactive statements would 'just know' when to execute based on their dependent variables being updated worried me that there would be cases when they executed more often than they needed to (think AngularJS digest cycle). 

After working with them for a while with lots of console logs I was pleasantly surprised to find they were generally not executing any more than necessary. Reactive statements will run whenever any of their dependent variables change, with Svelte automatically figuring out which order to run the statements in based on the dependent variables. Eg. in the above, `$: z = x + y;` must run before the `console.log` statement, as the console log depends on `z`.

Reactive statements actually work in a similar way to how a spreadsheet updates calculated cells based on the values of their dependent cells. [This video](https://www.youtube.com/watch?v=AdNJ3fydeao&ab_channel=YouGottaLoveFrontend) goes into this in some more depth, as well as showing you what the compiled JavaScript looks like.

## Minimal amount of JS shipped

Because Svelte compiles your components to Vanilla JavaScript there is less junk.js to be minified and shipped with your source code. Furthermore, if you do not use a framework feature (such as animations) then you are guaranteed that there will be no extraneous animation related code shipped. This is the theory at least, and it seems plausible for small and medium sized apps. 

My intuition was that if you were to reach a critical mass of source code it would be better (both in terms of download and browser scripting time) to use the React approach of shipping more framework code and less (due to the optimisations which shipping this framework code allows for) source code. After searching around online I found [some research](https://github.com/halfnelson/svelte-it-will-scale) which suggests that my intuition was correct, but that ending up with a larger bundle size in Svelte is unlikely unless your app is very large and uses no code splitting.

## Stores (good for small/medium size projects)

Stores are a lightweight state management system which come with Svelte. A store can be writable or readable:

```js
// Writable store
let myNumber = Writable(10);

myNumber.set(5); // Update the store value

// Readable store
let myConst = Readable(7);
// Derived store
let myNumberDoubled = Derived(myNumber, $myNumber => $myNumber * 2);
```

We can then either manually to a store with the subscribe method:

```js
myNumber.subscribe(n => console.log(n)); // 5
myNumberDoubled.subscribe(n => console.log(n)); // 10
myConst.subscribe(n => console.log(n)); // 7
```

However, inside of a `.svelte` component file it is possible to obtain the current value of a store by prefixing the store name with a `$`:

```svelte
$: {
  console.log($myNumber); // 5
}
```

The simplicity of this type of state management is nice on a smaller project, but I find myself having to add a lot of `console.log` statements to the codebase for debugging. 

On a larger project I would prefer a state management tool which has slightly more structure to it and can be debugged through the [Redux Devtools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd). I found a [possible solution](https://github.com/unlocomqx/svelte-reduxify) to this that I have yet to try, but it would be nice if it was easier to debug stores out of the box.

It is however worth noting that Svelte is compatible with any other solution which implements its store contract (including RxJS), so it is likely that we will see more state management solutions for Svelte emerging.

## Accessibility

Svelte has built in checks which warn you of any accessibility issues in your templates such as images with missing alt tags. This is a nice touch, in my opinion helping make websites accessible should be a core feature of a frontend framework.

## JavaScript inside of HTML, not HTML inside of JavaScript

This is more personal preference, but I like the fact that Svelte (similar to Vue) is essentially a superset of HTML, with the JavaScript and CSS included within the same file in `<script>` and `<style>` tags respectively.

# Reasons not to use Svelte

While working with Svelte is a mostly positive, there have been some pain points too.

## Tooling

Svelte's tooling is not great, this may be in part due to it being a newer framework.

A standard Svelte project can be created using `degit` as follows:

```bash
npx degit sveltejs/template my-svelte-app
```

This generates a Svelte project which uses `rollup` as the bundler - there is also a webpack template available. There is a script for converting your project to TypeScript, but this is about as much help as you will get with your project configuration.

Want to set up linting? You will have to try and configure it yourself, it also seems that [ESLint is not compatible with Svelte components which use TypeScript](https://github.com/sveltejs/eslint-plugin-svelte3/issues/68).

Want to generate bundles with file hashes? You'll have to spend a couple of hours messing around with your rollup config.

## Debugging

The nicest thing about Redux based state management is that you can debug your application through the Chrome Redux dev tools extension, rather than adding lots of console logs to your code. 

While the Svelte devtools are somewhat useful for checking component state, they do not allow you to examine the state of Svelte stores. I consider this a major issue for large applications.

## Ecosystem

There are no as many UI kits and other libraries out there for Svelte as there are for Angular/React/Vue. 

This doesn't seem like a huge issue for the smallish applications I have used it for as there is enough to get by, and it goes some way to defeating the point in using a lightweight solution like Svelte if you then pull in a whole bunch of heavy libraries. Nonetheless I think for a larger app I would choose a framework with a bigger ecosystem.

# Sapper, Svelte Native

It is possible to create native apps using Svelte Native, which is based on NativeScript. You can also do server side rendering and routing using Sapper. I haven't used either of these (yet) so can't comment as to what they are like.

# Conclusion

Svelte is an excellent choice for a small or medium sized app - reactive statements and stores are a unique and make it easy to build lightweight reactive apps. It feels significantly easier to build user interfaces with Svelte than with any other framework I have used.

For a very large app Svelte is probably not such a good choice - you may struggle with its relatively hard to organise, hard to debug state management solution. You may also find it has lack of choice of libraries due to its smaller ecosystem.

And unfortunately whatever size your app is, you are going to have to spend time messing around configuring your bundler.

# Additional Resources

Whilst learning Svelte, I found the following courses to be very useful and you may do too:

<figure>
  <a href="https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.3181236&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fsveltejs-3-and-sapper-projects%2F">
      <img src="https://img-a.udemycdn.com/course/480x270/3181236_6910_2.jpg?xBe3TB0tqXIyopcYeUaHIsG8Nnzm66YKQyZ_a66RqNN1ln_yB2r8wEU6wWsaCG86lnlB2eGQxxb98nT45bYr3taiGk0zZ2iDJeW6FTOHjJvSOxRtO7sPhY61qiNw7GcT"/>
      <figcaption>Svelte. js 3 and Sapper Projects</figcaption>
  </a>
</figure>

<figure>
  <a href="https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.2606044&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fsvelte-tutorial-and-projects-course%2F">
      <img src="https://img-a.udemycdn.com/course/480x270/2606044_64c5_2.jpg?Z948i5zJMKYmEMDJUxDQCxH80M0V9Hw2wt6SL4Tpx3tKsPz8ovmOVMX_ivb8LzW9jxrXljsf355Qd0HM4ZO9LSGh08LTRgNChl1UpsjIfQDZBuAN77lBURsC6NAk2Qt2"/>
      <figcaption>Svelte Tutorial and Projects Course</figcaption>
  </a>
</figure>

Additionally, the following resources were useful for the creation of this blog post:

* [Svelte Tutorial](https://svelte.dev/tutorial/basics)
* [React vs. Svelte: The War Between Virtual and Real DOM](https://blog.bitsrc.io/react-vs-sveltejs-the-war-between-virtual-and-real-dom-59cbebbab9e9)
* [Rich Harris - Rethinking reactivity](https://www.youtube.com/watch?v=AdNJ3fydeao&ab_channel=YouGottaLoveFrontend)