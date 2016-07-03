---
title: 'Using cookies with browser-side GitHub auth'
contributors: [sethvincent]
topics: [javascript, authentication, github, github-auth]
description: "Allow GitHub login on your static sites."
published: true
---

Building applications on top of the GitHub API often requires authenticating users, and in this tutorial we'll cover tracking a user's access token with cookies.

## Prerequisites

This tutorial builds on a previous tutorial in the series. First read and work through [Using gatekeeper to authenticate to GitHub with browser-side JavaScript](/posts/gatekeeper-for-authenticating-with-github/).

## Setting up your project

Navigate to the project directory for the code from the previous tutorial.

To see the code as it was at the end of that first tutorial, check out the post-1 branch of this repository: [github.com/sethvincent/github-auth-example/tree/post-1](https://github.com/sethvincent/github-auth-example/tree/post-1)

## Start the gatekeeper server

Make sure to remember to start your gatekeeper server with this command:

```
node gatekeeper/server.js
```

## New dependenciy: cookie-cutter

We'll use the module [cookie-cutter](http://npmjs.org/cookie-cutter) to manage the cookies in out application.

Install it with npm:

```
npm install --save cookie-cutter
```

Require the module at the top of the index.js file:

```
var cookies = require('cookie-cutter')
```

## Saving the auth token

We will start adding to the `start` function to first check if there's already a cookie with the access token, and if not, get the token or render the "log in with GitHub" link.

Our previous `start` function looked like this:

```
function start () {
  var code = getCode()

  if (code) {
    getToken(code, function (err, token) {
      getProfile(token, function (err, profile) {
        renderProfile(profile)
      })
    })
  } else {
    renderLink()
  }
}
```

We'll revise it to look like this:

```
function start () {
  var code = getCode()
  var token = cookie.get('github-auth-example')

  if (token) {
    getProfile(token, function (err, profile) {
      renderProfile(profile)
    })
  } else if (code) {
    getToken(code, function (err, token) {
      cookie.set('github-auth-example', token)
      window.location = window.location.origin
    })
  } else {
    renderLink()
  }
}
```

Our revised function has two major changes: checking for the token in a cookie and setting the cookie with the token after receiving the request that includes the code from GitHub's authentication flow.

### Checking for the session with the access token

This line gets the token if it is available:

```
var token = cookie.get('github-auth-example')
```

If it is available, we get the GitHub profile of the user:

```
if (token) {
  getProfile(token, function (err, profile) {
    renderProfile(profile)
  })
}
```

If the token is not available, we check to see if this is the request that was made by google that includes the `code` querystring.

### Setting the cookie with the access token

If the request did include the `code` querystring, we get the token using that code, set the cookie, then redirect the browser to the app's url without the `code` querystring:

```
else if (code) {
  getToken(code, function (err, token) {
    cookie.set('github-auth-example', token)
    window.location = window.location.origin
  })
}
```

Then, like before, if the request does not include the `code` querystring or there is not already a cookie with the token, the login link is rendered:

```
else {
  renderLink()
}
```

### Try it!

Run `npm start`, go to `http://localhost:9966` and click the login link.

You should now be able to refresh the browser after logging in and the site will remember you!

But now, we need to be able to log out.

## Logging out

To add a logout link, we'll revise the `renderProfile` function.

Previously the `renderProfile` function looked like this:

```
function renderProfile (profile) {
  var p = document.createElement('p')
  p.innerHTML = profile.name
  document.body.appendChild(p)
}
```

We'll revise the function to add the log out link:

```
function renderProfile (profile) {
  var p = document.createElement('p')
  p.innerHTML = profile.name
  document.body.appendChild(p)

  var logout = document.createElement('a')
  logout.href = '#'
  logout.innerHTML = 'log out'
  document.body.appendChild(logout)

  logout.addEventListener('click', function (e) {
    e.preventDefault()
    cookie.set('github-auth-example', '')
    window.location = window.location
  })
}
```

First we add the log out link to the document body:

```
var logout = document.createElement('a')
logout.href = '#'
logout.innerHTML = 'log out'
document.body.appendChild(logout)
```

Then we add an event listener to the logout link that resets the cookie and refreshes the page:

```
logout.addEventListener('click', function (e) {
  e.preventDefault()
  cookie.set('github-auth-example', '')
  window.location = window.location
})
```

Go back to the browser and try out the log out button. Success!

## Example source on GitHub

All the source code for this example is available on GitHub: [https://github.com/sethvincent/github-auth-example](https://github.com/sethvincent/github-auth-example)

Feel free to make pull requests, fork the code, etc.!

## This post is open source
Spot a typo? Have a great addition to make? [Send a pull request](https://github.com/cityarcade/makerlog.org/blob/master/_posts/2015-09-03-using-cookies-with-browser-side-github-auth.md)