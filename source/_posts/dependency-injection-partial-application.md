---
title: Dependency Injection with Partial Application in Javascript
date: 2018-05-30
author: Hector Yeomans
author_id: hyeomans
tags:
  - javascript
  - functional-programming
---

In statically typed languages like C# or Java, you become familiar with the concept of dependency injection. There is even a group of practices all assembled in something called an IoC container, IoC standing for Inversion of Control. The first time I heard of DI (dependency injection) in Javascript world was with AngularJs. I'm talking about Angular 1, and it looks something like this:

<!-- more -->

```javascript
angular
  .module("myModule", [])
  .factory("serviceId", ["depService", function(depService) {}]);
```

There is an excellent talk by Mark Seemann on dependency rejection[1]. The first half of Mark's presentation talks about how partial application is a non-functional way of doing dependency injection in functional languages. Dependency rejection title comes from the idea of pure functional languages, where partial application is not sufficient to do dependency injection of non-pure function calls. Dependency rejection is funny because Mark wrote a whole book about Dependency Injection[2]. Chapters one to nine of Mark's book talk thoroughly about dependency injection, and personally, my favorite chapters are seven, eight and nine. In those three chapters, you understand three significant characteristics of an IoC container: **Object composition**, **Object Lifetime**, and **Interception**.

I would like to tackle **Object composition** in this post, but how do you translate object composition to partial application?

Before answering that question, there was an image that struck a chord in Sandi Metz book:

| ![](https://www.safaribooksonline.com/library/view/practical-object-oriented-design/9780134445588/graphics/04fig01.jpg) | 
|:--:| 
| *Practical Object-Oriented Design in Ruby - Chapter 4 Creating Flexible Interfaces [3]* |

Imagine each of the nodes in that graph is a class, and each edge is a call between classes. Now imagine you get to choose which codebase you can work on, which one would you pick? I want to believe most of you will select the cleaner version with fewer edges. A new question arises, how do you achieve that in a code base?

I will walk you through from an OOP language to a partially applied example. For this, I will pick Ruby and Javascript because those are the languages I've been working on lately but hopefully, the concepts are clear that you can apply them to the programming language you're using. If you're interested in how to do it in Haskell or PureScript, I would recommend following Mark Seeman talk on F# [1]. 

Imagine you need to integrate a new reservation system to your application. Thankfully, there is an existing SDK to handle this for you, so you add the gem to your Gemfile and bundle install, after that, you get something like:

```ruby
require 'external-api' #1

class ReservationsManager
  def initialize(token)
    @client = ExternalApi::Reservations.new(token) #2
  end

  def retrieve_reservations(start_date, end_date)
    @client.reservations_for(start_date, end_date) #3
  end
end

##### In another portion of your Rails or Ruby application
manager = ReservationsManager.new('my-token')
now = Time.now
eight_hours_before = now - (60 * 60 * 8)
manager.retrieve_reservations(now, eight_hours_before)
```

Nothing you haven't seen in the OOP world of Ruby:

1) Adding the required gem to have access to `external-api` SDK.
2) Creating a new instance of the client
3) Calling the client for reservation on specific date. Disregard any error cases.

By making this class, you made the first iteration of the tangled version of the graph. There are two classes right now, and a bidirectional edge has been formed. @client and ReservationManager are tightly coupled. I won't go into all the possible cases of how this might change in the future, but one thing is sure in software development, change is constant.

What we're interested is in how do we make this decoupled. The idea is simple yet powerful. Take a moment to see the angular example. Instead of calling the dependency service directly into our angular modules, you had to pass it as part of the function arguments. Same idea here:

```ruby
class ReservationsManager
  def initialize(client)
    @client = client
  end

  def retrieve_reservations(start_date, end_date)
    @client.reservations_for(start_date, end_date) #3
  end
end

##### In another portion of your Rails or Ruby application
require 'external-api' #1
client = ExternalApi::Reservations.new(token)

manager = ReservationsManager.new(client) #2

now = Time.now
eight_hours_before = now - (60 * 60 * 8)
manager.retrieve_reservations(now, eight_hours_before)
```

1. SDK is now required in the application root. Think of where your application starts.
2. `client` is now injected into the constructor.
3. `retrieve_reservations` interface has not changed.

The change is simple. However, you can now mock the injected dependency easily, I know you could that before, but now the dependency is explicit. Another benefit is that if you ever want to change the client provider for reservations. The only requisite is that it has to comply with the same `reservations_for` interface. I honestly thought that didn't happen in production, but I learned my lesson when I had to migrate from Braintree to Stripe, or when migrating from Incomm to another gift card provider or that one where we had sendgrid, but we wanted amazon ses.

> Hector I came here for partial application.

So now the Javascript version of this:

```javascript
function retrieveReservations(token, startDate, endDate, cb) {
  $.ajax({
    url: `/reservations?start_date=${startDate}&endDate=${endDate}`,
    headers: {
      'X-AUTH-TOKEN': token
    }
  }).done(cb);
}

//In another portion of your javascript app:

function handleDone(response) { console.log('I got a response', response); }

const today = new Date();
const yesterday = new Date(Date.now() - 864e5);
const token = 'my-public-token';

retrieveReservations(token, today, yesterday, handleDone);
```

Unfortunately, for the front-end version there wasn't an SDK. So we're stuck with the jQuery version. Let's apply some partial application to it:

```javascript
function ajaxClient(token, startDate, endDate, cb) { //1
  $.ajax({
  url: `/reservations?start_date=${startDate}&endDate=${endDate}`,
  headers: {
  'X-AUTH-TOKEN': token
  }
  }).done(cb);
}

function retrieveReservations(client) {//2
  return (token, startDate, endDate, db) {
    return client(token, startDate, endDate, cb);
  }
}

//In another portion of your javascript app:
function handleDone(response) { console.log('I got a response', response); }

const withAjax = retrieveReservations(ajaxClient);//3
const today = new Date();
const yesterday = new Date(Date.now() - 864e5);
const token = 'my-public-token';

withAjax(token, today, yesterday, handleDone);
```

1. You define a simple function to make an AJAX call
1. You partially apply the client to `retrieveReservations` which is saved in the closure.
1. You __inject__ the ajax client, and the signature of the function stays the same as the first example.

What would you make this more difficult for new developers? Like everything the complexity has increased a little bit, but the ability to inject new clients has been obtained. Imagine that now you want to use `fetch` instead of `$.ajax` to make your requests:

```javascript
function fetchClient(token, startDate, endDate, cb) { //1
  return fetch({
    url: `/reservations?start_date=${startDate}&endDate=${endDate}`,
    headers: {
      'X-AUTH-TOKEN': token
    }
  }).then(r => r.json()).then(cb);
}

function handleDone(response) { console.log('I got a response', response); }

const withFetch = retrieveReservations(fetchClient);//2
const today = new Date();
const yesterday = new Date(Date.now() - 864e5);
const token = 'my-public-token';

withFetch(token, today, yesterday, handleDone); //3
```

1. Define a wrapper for your `fetch` client.
1. Inject the client into `retrieveReservations`.
1. Call with same function arguments, which didn't change.

## Conclusion
Partial application becomes very handy as your application grows. One point I didn't make, is the ability to isolate your tests and mock whatever dependencies you have. If you want to reduce boilerplate for partial application or currying I would recommend Ramda.js[4] 

* [1] https://www.youtube.com/watch?v=cxs7oLGrxQ4
* [2] https://www.manning.com/books/dependency-injection-in-dot-net
* [3] http://www.poodr.com/
* [4] https://ramdajs.com/
