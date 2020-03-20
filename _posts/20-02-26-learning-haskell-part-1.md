---
layout: post
title: "Haskell initial work"
image: ""
published: false
---

// TODO: intro

# Installation/setup

The best way to get started with Haskell is to install [Haskell Stack](https://docs.haskellstack.org/en/stable/README/). This is a package which contains the Global Haskell Compiler, as well as a set of tools for developing Haskell applications. 

Haskell Stack can be installed by following the instructions given [here](https://docs.haskellstack.org/en/stable/README/#how-to-install).

# GHCi

Haskell source code can either be entered into an interactive console named [GHCi (Global Haskell Compiler interactive)](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/ghci.html), or entered into source code files. 

Usually, the best way to write applications is to write your code in source code files. However, a good way to have a first play around with Haskell is vis GHCi.

Once you have installed the Haskell Stack, you can run GHCi from the command line:

```
stack ghci
```

TODO: what is prelude?

You can enter 2 types of information into GHCi:
* GHCi commands, these all begin with a `:` and are not part of the Haskell language, they are special commands for the GHCi software, eg. `:quit` will exit GHCi.
* Haskell Source Code, GHCi will compile and execute any source code given to it.


# Basic arithmetic

Arithmetic using GHCi - Evaluation.
Precedence, associativity
Prefix and Infix operators
Functions.

TODO: basics using ghci. Rules followed for precedence and associativity? Evaluation.

# Expressions

Expressions can be values, combinations of values or functions applied to values. Some examples of expressions are

```
1
```

```
3 * (4 + 5)
```

Expressions can be evaluated. When an expression is evaluated to the point where it can no longer be reduced, the expression is said to be in normal form.

# Parenthesese and $

TODO:

# Source code files

* Generally you will want to write code into source code files. I am using Visual Studio code as my editor to do this, there are various useful plugins such as Haskell Syntax highlighting which will help you out.

# Useful Resources

* Haskell Book
