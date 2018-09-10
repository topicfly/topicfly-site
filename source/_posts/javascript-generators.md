---
title: Javascript Generators
date: 2018-09-10
author: Hector Yeomans
author_id: hyeomans
tags:
  - javascript
  - es6
  - generators
---

![](http://res.cloudinary.com/diesu5vtq/image/upload/c_scale,e_auto_color,w_2087/v1536551949/jason-blackeye-212990-unsplash.jpg)

## Background

Javascript had a significant revision that completed on June 2015. This revision became known as ECMAScript 2015 or ES6. 

One of the features added at ES6 was generators.

<!-- more -->

## What are they?

Generators are functions that are special in Javascript. You can pause their behavior, and when halted you can send back arguments. This combination of stopping and sending back information allows two-way communication.  

You might have heard of `async` & `await` in Javascript. You can simulate async/await with generators with the difference that generators came before async/await was a standard into ECMAScript.

## Common terminology

* run-to-completion: This means that a function is guaranteed to run from start to finish without anything else interrupting.
* yield: This is the keyword that will make a generator function to pause.
* promise: Usually generators work with the Promise standard. Promises are functions that return another function which might contain a value or throw an exception.

## Why use them?
I think there are three significant players in the generator world: [`redux-saga`](https://redux-saga.js.org/), [`co`](https://www.npmjs.com/package/co) and the new implementation of [async iteration](https://github.com/tc39/proposal-async-iteration).

If you work with react/redux, you might step into a project that has redux-saga. Generators were popular between 2014-2016 when the spec for `async/await` wasn't ready, and some projects still use `koa` based on generators.

## Quick example

Let's see a sample generator function:


<iframe width="100%" height="300" src="//jsfiddle.net/u4tym1ar/26/embedded/js,result/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>

A lot is going on here. Here is the breakdown:

1. A Generator `function` is declared with a `*` somewhere in between function and the function name.
1. `yield` is the reserved keyword that pauses the function indefinitely.
1. Generator functions can receive values once unpaused.
1. Generator functions do not run when called.
1. Calling `next` is twofold: it runs until it meets yield and also returns an object with an interface: `{ value: [value], done: [boolean] } `
1. Generator function recognizes that hasn't run to completion hence the boolean flag being false.
1. Calling `next` with an argument allows the consumer of a generator function to "talk back" to the producer function. 
1. Remember that calling `next` always returns an object, in this case, the generator function has run to completion.
1. Generator function has ran all statements, so done is now `true`.

## Conclusion

Even though you saw a rather simple example, there is a lot to unpack when working with generator functions:

* You enter the realm of producer/consumer interaction.
* You can have two-way communication between producer and consumer.
* Pausing will be the most common scenario.
* You could have logic, that makes the generator function, behave differently on each call to `next(param)`.

Promises weren't touched in this example but will be covered in future posts. Generator functions work in both asynchronous and synchronous modes. Async/await is the key of future usage of generators.

<sub><sup>Photo by Jason Blackeye on Unsplash</sup></sub>