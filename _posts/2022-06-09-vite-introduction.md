---
layout: post
title: "Vite introduction"
image: ""
published: false
---

# What is Vite?

Vite is a next generation frontend tooling solution which has a very fast dev server by taking advantage of native ES modules and dynamic ES module imports (`import`/`export` and `await import('my-dep')`).

## Vite dev server

* Only dependencies which will not be edited by the developer (eg. node_modules) are bundled. Bundling is done using ESBuild, an extremely fast modern JS bundler.
* Instant server start - file serving over native ES Modules.
* Hot module replacement - when you change a file Vite doesn't have to reload the whole page, just fetch the modified ES module.

## Vite production build

* Optimised build using Rollup - minimal config required
* Many existing Rollup plugins can be used if needed - https://vite-rollup-plugins.patak.dev/

## Advantages over Webpack, Rollup

* Very fast dev builds compared to 'traditional' bundlers.
* Much nicer API for the config file - config is typed and well documented.
* Rollup does not support Hot Module Replacement, Webpack does support it but badly.

## Advantages over Snowpack

* Easy to configure production build - 'batteries included' approach based on Rollup build.

# Who is using Vite

* Crown V2
* Sveltekit
* Other common frontend JS frameworks - React, Vue, etc...

# Useful links

* https://vitejs.dev/guide/why.html#the-problems
* https://esbuild.github.io/
* https://caniuse.com/es6-module-dynamic-import

