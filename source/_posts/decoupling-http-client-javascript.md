---
title: Decoupling HTTP clients React-Native 
date: 2018-05-15 
author: Hector Yeomans
tags:
  - javascript
  - react-native
  - node.js
---

Recently I was working on a React Native application. As is usual with a mobile client I had to write some
http calls to a back-end server. I started using a pretty standard pattern in my code:

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

Nothing complicated, you problably have done this a couple or dozens of times. It wasn't until I finish maping
all endpoints where I realized that it will be cool if I could use all files in a Node.js application.

The only problem is that `fetch` comes by default on `create-react-native-app` and not by default on Node.js. The easy way would be to add something to `node-fetch` to my dependencies and call it a day. However I just didn't want to add a new dependecy because I usually use `request-promise` or `axios` on the back-end. Also, it's not only about the library that I use in the front-end or back-end, I just wanted to have the client be decoupled of the means of transportation.

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

it does not take in consideration body vs data
