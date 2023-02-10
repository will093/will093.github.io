---
layout: post
title: ""
image: ""
published: false
---

# Chapter 1

program - application without testing/docs which runs on the author's system/local environment.
programming product - program + documentation, testing, maintenance, generalisation across environments
programming system - program + integration with other programs as needed per business requirements

Takes 3x work to go from a program to a programming system or programming product

# Chapter 2

Software tasks cannot easily be partitioned - more team members require more exponentially more communication as the team size grows. No such thing as a 'man month' (amount of work done by 1 person in 1 month).

# Chapter 3

Recommends small teams which each focus on a specific module of the application, each member having their own responsibilities, and only 1 or maximum of 2 people actually writing the code. Hierarchical team with 1 person in charge.

# Chapter 4

Conceptual integrity - architecture and implementation should be separated

Architecture - technical specification, ie. design of how the system should work from the user's point of view.
Implementation - building of the program based on the architecture/specification.

Architecture should be done by a very small number of people to ensure that the overall philosophy and concepts of the program are aligned.

# Chapter 5

Second system effect - architect often overdesigns the second iteration of a system.

* Architect has a tendency to build a simple first system with minimal embellishments. 
* Second system is often subject to bloat from too many embellishments and features. 
* Third system onwards, the architect knows the generalisable characteristics of the type of system they are designing.

Solution for the architect is simply self discipline?

Is this applicable to software products using an agile methodology today? MVP product is the initial system, subsequent iterations on this are small.

Could this be applied to the second website/webapp built with a specific architecture?

# Chapter 6

Written specifications/manuals should be written by software architects. What the user sees, all user interfaces.

* Presumably for a service 'what the user sees' denotes the public API of that service. 

Formal definition - precise, gaps and omissions are conspicuous, but incomprehesnable.
Prose definititon - show key structural principles, emphasize certain principles, give examples (but lack precision).


Formal definitions often contain implementation details, but really they should not, they should only describe externals.

An implementation can serve as a formal definition, but this can be bad because:

* Existing implementation can be wrong for some inputs which weren't considered, or could have some side effects which we would not really want to copy to a new system.

* Architects, implementers, users (and oither stakeholders) should join weekyl meetings/conferences and propose problems or changes.

* Multiple implementations happening simultaneously can particularly help to enforce a written specification (I suppose ECMAscript is an example of this)