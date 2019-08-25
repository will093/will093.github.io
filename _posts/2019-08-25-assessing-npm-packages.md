---
layout: post
title: "The questions that you should be asking yourself when you install an npm package"
image: "/images/assessing-npm-packages/question.jpg"
published: true
versions: npm 6.x.x
---

For any JavaScript project, whether it be frontend or backend, we usually rely on a number of third party, open source npm packages - these might be frameworks, utility libraries or build tools.

This is because, understandably, we do not want to 'reinvent the wheel'. Often using a framework and a few well selected third party utility libraries will speed up the development of and increase the maintainability of an application. 

However, when not used with discretion, these open source packages can be harmful to your project - you must remember that each package that you decide to use can introduce security, maintainability, performance and legal issues.

In this article I will go through the questions that you should be asking yourself when deciding whether you should use a particular npm package in your project.

# 1) Is the package being actively maintained?

It is worth considering whether a package you plan to use is being maintained. 

Some open source packages are maintained by companies or organisations, and others by individual developers. Of course you cannot control whether a package will continue to be maintained going forwards, but there are some good indicators that you can look for as to whether a package is being maintained currently and whether it will be in the near future.

* **Look at how long ago the most recent version was published on npm**

    Search for the package on <https://www.npmjs.com>, and you will see how long it was since the a version of the package was last published. If the last version was published several months ago or longer, then it could be that the package is no longer being maintained.

    ![Last publish: 3 days ago](/images/assessing-npm-packages/npm-last-publish.png)

* **Look at the number of weekly downloads on npm**

    A large number of weekly downloads is a good indication as it means that people are using the package, which is an indication of a certain level of quality.

    ![Weekly Downloads](/images/assessing-npm-packages/npm-weekly-downloads.png)

    Be wary, however, as it can sometimes be the case that a large number of people are using an unmaintained package. [Twitter Typeahead](https://www.npmjs.com/package/typeahead.js) is a good example of this - it is a package which was initially was maintained, but was at some point abandoned by the maintainers. Large numbers of people continued to use the package however, but for the last 4 years no maintenance has been done and no new versions published. 
    
    If we [look at the package on npm](https://www.npmjs.com/package/typeahead.js), it still has a reasonably large number of weekly downloads to this day.

* **Look at the number of open issues on Github, and whether these issues are being addressed and closed by maintainers.**

    It is not necessarily a bad thing for a package to have a large number of issues, especially if it is a large library or framework - this just means that people are using the package and reporting issues with it, which is a good thing. 

    What you should try to avoid is a package where maintainers are not addressing open issues and where very few issues have recently been closed. Again, [Twitter Typeahead](https://github.com/twitter/typeahead.js/issues) is a good example of this.

# 2) Does the package introduce a security risk?

Any package that you install may have **known** and/or **unknown** security issues associated with it. 

For **known** security issues, [npm audit](https://docs.npmjs.com/cli/audit) can help by letting you know of any security issues with a package or its dependencies - allowing you to either update the package or make the decision not to use it. 

For npm 6 and above, an audit is done when you install an npm package via `npm install <package-name>`, and can also be run against a package manually by running `npm audit` in the root folder of the package.

# 3) What kind of licensing does the package have?

There are a number of different licenses that an open source npm package may have. 

A lot of the time you will find that a package is completely free to use for both personal and commercial use; under licenses such as the Apache License 2.0 or the MIT License. 

At other times, however, you may find that a package requires licencing for commercial use, or has other strings attached. In the past, React - now licensed under MIT - was licensed under [BSD + patents](https://hackernoon.com/facebooks-bsd-patents-license-and-how-it-affects-you-66088e052845), which effectively said that you could not use it to build a product which could be seen to be competing with Facebook.

# 4) Will the package be flexible enough for my requirements?

Sometimes you can be fairly sure of the requirements for a particular feature, other times you may find that you or your client/employer doesn't know exactly what they want.

A good example of this would be if you were asked to implement a form with a datepicker, and could not use the native browser date picker, as it needed to have cross browser consistency.

You might decide to complete this task using [Tiny Date Picker](https://www.npmjs.com/package/tiny-date-picker) - this is a small, popular, lightweight npm package which provides datepicker and daterange picker UI components.

![Weekly Downloads](/images/assessing-npm-packages/tiny-date-picker.png)

However, you may later decide that it is important to you that your application is accessible/WCAG 2.0 compliant. With a third party UI component like this there is no way to modify the source code, and so there is no easy way to add the appropriate aria attributes or to make sure that the tab order and keyboard navigation behave as you require.

At this point, you would either have to switch to an alternative package which meets your requirements for accessiblity, fork the package you are currently working with and modify it to do what you require, or else implement something yourself.

# 5) How will the package affect the weight of my application?

This is particularly important for a frontend application, where every third party package that you add to your application is additional KBs that your user's will have to download.

You can get an idea of how much any given package is affecting the weight of your application by compiling your application with source maps, and then analysing these source maps with [source-map-explorer](https://www.npmjs.com/package/source-map-explorer). You can then make an informed decision as to whether the package is worth the extra KBs.

![Source map explorer](/images/assessing-npm-packages/source-map-explorer-2.png)

# 6) Does the package have a clean, well documented API?

If it takes you a long time to understand how a package works, learn its API and how to integrate it into your project, and it is not well documented, then this is a warning sign. 

Every other developer who works on the project will have to go through the same learning process that you did in order to be able to work on your project.

# 7) Could I easily implement this bit of functionality myself?

Sometimes you just require a small proportion of the functionality that a particular package provides, and you could write the code for this functionality yourself, in a very small amount of time. 

If you can write that functionality in a small amount of time, then it may be easier to maintain this small amount of code that you have written yourself than it is to maintain your project with an additional dependency.

For example, if you found that in your application you needed to split an array into a number of chunks, then you could do this with the `chunk` method from the npm package [lodash](https://www.npmjs.com/package/lodash). In the case where your application could benefit from a number of lodash functions, then this approach would make a lot of sense.

However, if this were the only method that you needed from lodash, then it might make more sense to implement this utility function yourself - [doing so is pretty easy](https://scotch.io/courses/the-ultimate-guide-to-javascript-algorithms/array-chunking#toc-looping-through-the-number-of-chunks
), and leaves you with one less package with the potential security, maintainability and licensing issues mentioned in this article (okay lodash doesn't have these concerns to such a degree as some other packages out there, but you get the point).

# Conclusion

These are the questions that you should be asking yourself every time you add an npm package to your project. 

In general I am a begin fan of reusable open source code and believe that there are many problems to which an open source npm package is the solution. However, there are also times when implementing your own solution is the right way to go; it is often easier than you might first think (and of course very rewarding).