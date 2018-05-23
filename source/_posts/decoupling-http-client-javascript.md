---
title: Decoupling HTTP clients React-Native 
date: 2018-05-15 
author: Hector Yeomans
tags:
  - javascript
  - react-native
  - node.js
---

Recently I was working on a React Native application. As is usual with a mobile client I had to write some HTTP calls to a back-end server. I started using a pretty standard pattern in my code:

<!-- more -->

```javascript
function login(email, password) {
  const payload = {
    method: 'POST',
    headers: {
      Accept: 'application/json'
    }
    body: JSON.stringify({email, password});
  };

  return fetch('https://myapi/login', payload);
}
```

Nothing complicated, you probably have done this a couple or dozens of times. It wasn't until I finish mapping all endpoints where I realized that it would be cool if I could use all files in a Node.js application.

The only problem is that `fetch` comes by default on `create-react-native-app` and not by default on Node.js. The easy way would be to add `node-fetch` to my dependencies and call it a day. However, I didn't want to add a new dependency, what if I want to use `request-promise` or `axios` on the back-end? **Also, it's not only about the library that I use in the front-end or back-end, I just wanted to have the client be decoupled of the means of transportation.**

So I came up with something like this:

```javascript
//login.js
function login(wrappedHttpClient) {
  return (email, password) => {
    const payload = {
      method: 'POST',
      headers: {
        Accept: 'application/json'
      }
      body: JSON.stringify({email, password});
    };

    return wrappedHttpClient('https://myapi/login', payload);
  }
}

//In another file

function wrappedAxiosClient(url, payload) {
  return axios(url, payload);
}

function wrappedFetchClient(url, payload) {
  return fetch(url, payload);
}
```

There is a problem with the previous setup. Let's take `node-fetch` and `axios` as an example. Both libraries return a promise once the request is made, nonetheless both have different payload requirements. What I mean by this is, take a look at the following code on how to make a POST request with both libraries:

```javascript
//With axios
axios(url, {
  method: "post",
  data: {
    firstName: "Fred",
    lastName: "Flintstone"
  }
});

//And now with fetch
fetch(url, {
  body: JSON.stringify(data),
  method: "POST" // *GET, POST, PUT, DELETE, etc.
});
```

In the case of `axios` you need a `data` property and for `fetch` you need a `body` property. So what I ended up doing is:

```javascript
//My Fetch wrapper
function wrappedFetchClient(baseUrl) {
  return {
    post
    //Same for all other methods
    //get,
    //put,
    //remove I don't call it DELETE because Javascript has issues with delete
  };

  function post(urlPath, body) {
    const payload = {
      body: JSON.stringify(body),
      method: "POST"
    };

    return fetch(`${baseUri}${urlPath}`, payload);
  }
}

//My axios wrapper
function wrappedAxiosClient(baseUrl) {
  return {
    post
    //... all other HTTP methods
  };

  function post(urlPath, body) {
    const payload = {
      body: body, //axios does not need to stringify as fetch
      method: "POST"
    };

    return axios(`${baseUri}${urlPath}`, payload);
  }
}
```

You might think is overkill to wrap your HTTP client, but remember it's only four (or seven if you're super pro) methods that you need to wrap and the benefit is that now you have the flexibility to use your API client code in both the front-end, back-end or react-native.
