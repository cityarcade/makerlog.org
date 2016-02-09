---
---

# Authenticating with GitHub using browser-side JavaScript

[github-secret-keeper](https://github.com/HenrikJoreteg/github-secret-keeper) is a node.js project for allowing users to login to GitHub from static sites. To complete the authentication, GitHub requires the `client_secret` of the API keys to remain secret, so it can't be sent with browser-side JavaScript. 

That's what github-secret-keeper is for. It's a simple server that has one job: accept requests with a GitHub authentication code, send the code & API keys to GitHub, and send back a response to your site with the access token for that user.

This chapter will walk you through setting up github-secret-keeper and using it to get the json object of the profile for users that log in to GitHub through your site.

## Prerequisites

You'll need [node.js](http://nodejs.org/download) and [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed on your computer before starting this tutorial.

You'll also need an account on [github.com](http://github.com).

## Setting up your project

Let's do some initial setup, including creating a new folder, a .gitignore file, a config.js file, a package.json file, and an index.js file for your browser code.

We'll also create an application on GitHub to get the API keys we'll need to make requests to the GitHub API. Let's start with that.

### Create an application on GitHub

If you haven't already, create an account on [github.com](http://github.com) and log in.

Click on your name at the top right of the screen, then click **Setting**.

![screenshot of settings link](images/settings.png)

Then click the **Applications** link in the menu on the left.

![screenshot of applications link](images/applications.png)

Next click **Developer Application**, and then **Register new application**.

![screenshot of register new application button](images/register.png)

Finally, fill out the **Application name**, **Homepage URL**, and **Authorization callback URL** for your application.

![screenshot of new application form](images/form.png)

You can use these settings for your new application:

- Application name: `github-login-dev` (or whatever you want to call it)
- Homepage URL: `http://127.0.0.1:9966`
- Authorization callback URL: `http://127.0.0.1:9966`

Click the **Register application** button and you're finished registering the application!

On the next screen you'll see the **Client ID** and **Client Secret** that we'll need later in the tutorial. Keep this browser tab open for later.

### Create the project folder

To create the folder, open a terminal window, navigate to a directory that you use to store projects, and run this command:

```
mkdir github-login
```

You can replace `github-login` with whatever you want to call your project.

Next, change directory into the project using the `cd` command:

```
cd github-login
```

### Create the .gitignore file

Using your favorite text editor create a file named `.gitignore`. For this tutorial I'll use `nano`, a simple command-line text editor:

```
nano .gitignore
```

We'll edit this file to exclude common files from being included in the git repository of the project. Add these lines to the file:

```
npm-debug.log
node_modules
github-secret-keeper
```

**Tips for using `nano`:** To save a file, press the keys `Control` and `O` at the same time, then press `Enter`. To exit nano, press the keys `Control` and `X` at the same time.

The .gitignore file now excludes the error log `npm-debug.log` that npm creates when an error occurs, the `node_modules` folder where all dependencies are stored, and the `github-secret-keeper` folder. During development I like installing github-secret-keeper inside of the project directory for simplicity, but we don't want to check the github-secret-keeper code into our project, so we exclude it with .gitignore.

### Create the config.js file

The config.js file is where we'll store global variables like the url of the github-secret-keeper server, the redirect uri that we need to send to GitHub, and the `client_id` part of the GitHub API keys of the application.

Using `nano`, create the config.js file:

```
nano config.js
```

You'll find that you'll eventually need two sets of this configuration: one set for development, and one set for production. To prepare for this, let's set up the config.js file like so:

```js
var config = {
  development: {},
  production: {}
}

module.exports = config[process.env.NODE_ENV]
```

For now we'll just fill in the `development` object. Later when you deploy your project you'll want to create a second application on GitHub and fill in the `production` object with the appropriate values.

Here's the config.js file updated with the `development` config:

```js
var config = {
  development: {
    client_id: 'your client id',
    redirect_uri: 'http://127.0.0.1:9966',
    ghAuthHost: 'http://127.0.0.1:5000'
  },
  production: {}
}

module.exports = config[process.env.NODE_ENV]
```

### Create the index.js file

Using `nano` again, let's create the index.js file:

```
nano index.js
```

To start, we'll just require the config.js file:

```
var config = require('./config')
```

### Create the package.json file

To create the package.json file, run the following `npm` command:

```
npm init
```

This will ask you a few questions about your project. For this tutorial you can hit `Enter` to use the default values for all the questions.

## Installing github-secret-keeper

To install github-secret-keeper we'll complete a few steps: cloning the repository, installing github-secret-keeper's dependencies, and setting up the config for github-secret-keeper.

### Cloning the repository

Clone the github-secret-keeper repository with the following command:

```
git clone https://github.com/HenrikJoreteg/github-secret-keeper.git
```

This will create a `github-secret-keeper` folder in your project directory.

Change directory into github-secret-keeper with the `cd` command:

```
cd github-secret-keeper
```

### Install github-secret-keeper's dependencies

Use the `npm install` command to install all of github-secret-keeper's dependencies:

```
npm install
```

### Edit github-secret-keeper's env.json file

github-secret-keeper also has a config file, but this one is called **env.json**.

Use `nano` to edit the file.

```
nano env.json
```

You should see empty curly brackets:

```
{}
```

Edit the file to add your client id as the key, and the client secret as the value:

Like this:

```js
{
  "client id": "client secret"
}
```

An example might look like this:

```js
{
  "e17a3afcc2c7691bfd12": "ba886cb219869877bd59ba9371af6aeffa959634"
}
```

### Start the github-secret-keeper server

You can start the github-secret-keeper server with this command:

```
node server.js
```

If you are in the root directory of your project instead of inside the github-secret-keeper directory, you can start github-secret-keeper with a slight revision:

```
node github-secret-keeper/server.js
```

After starting the server you should see output similar to this:

```
token server started at http://0.0.0.0:5000
```

## Start your app!

Alright, we've got github-secret-keeper all set up. Now we can get to the fun part: authenticating users.

In this section we'll install project & development dependencies, set up some npm scripts, and write some JavaScript for making requests to github-secret-keeper and GitHub.

### Install development dependency: budo, envify, browserify

We'll use a tool called [budo](http://npmjs.org/budo) to serve our project to the browser. It's convenient because it'll serve an html file for us if we haven't already created one for the project.

We'll also use [envify](http://npmjs.org/envify), which allows us to use environment variables in our browser-side code, enabling our cool development/production trick in config file.

[Browserify](http://npmjs.org/browserify) is what we'll use to bundle our browser-side JavaScript that uses CommonJS/Node style require statements.

To install the dependencies, run the following `npm` command:

```
npm install --save-dev budo envify browserify
```

### npm script: `npm start`

We'll use budo to serve our application, and to run it, we'll add a property to the `scripts` section of the package.json file of our project.

In the project directory, open up the package.json file using `nano`:

```
nano package.json
```

The contents should look something like this:

```js
{
  "name": "github-login",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "browserify": "^11.0.1",
    "budo": "^4.2.1",
    "envify": "^3.4.0"
  }
}
```

Replace the `test` property in the `scripts` object with a `start` property that runs the budo command:

```js
"scripts": {
  "start": "budo index.js:bundle.js --live -- -t [ envify --NODE_ENV development ]"
},
```

The `--live` part of the command makes it so budo watches for changes, and when you save a file, re-bundles the JavaScript and restarts your browser window.

The `-t [ envify --NODE_ENV development ]` part of the command is what tells browserify to make `process.env.NODE_ENV` equal the value `development`.

### npm script: `npm run bundle`

Next let's add a `bundle` script. This is what you will use to bundle your code for production.

The `bundle` script should look like this:

```js
"bundle": "browserify index.js -t [ envify --NODE_ENV production ] -o  assets/bundle.js"
```

Note that now the `[ envify --NODE_ENV production ]` section of the command will make `process.env.NODE_ENV` equal the value `production`.

The `scripts` object should now look something like this:

```js
"scripts": {
  "bundle": "browserify index.js -t [ envify --NODE_ENV production ] -o assets/bundle.js",
  "start": "budo index.js:bundle.js --live -- -t [ envify --NODE_ENV development ]"
},
```

Let's make sure that the `npm start` script is working.

Open up the index.js file and add a `console.log` statement that logs the `config` object.

Like this:

```js
var config = require('./config')

console.log(config)
```

Next, run `npm start`.

You should see output similar to this:

```
> github-login@1.0.0 start /Users/sethvincent/workspace/makerlog/examples/github-login
> budo index.js:bundle.js -- -t [ envify --NODE_ENV development ]

{"time":"2015-08-25T01:20:16.285Z","hostname":"pizza-2.local","pid":49491,"level":"info","name":"budo","message":"Server running at http://localhost:9966/","type":"connect","url":"http://localhost:9966/"}
{"time":"2015-08-25T01:20:25.489Z","hostname":"pizza-2.local","pid":49491,"level":"info","name":"budo","url":"/","type":"generated"}
{"time":"2015-08-25T01:20:25.699Z","hostname":"pizza-2.local","pid":49491,"level":"info","name":"budo","url":"/bundle.js","type":"static"}
```

Now go to [http://localhost:9966](http://localhost:9966) in your browser and open up the JavaScript console in your browser's developer tools. You should see the config object logged like this:

![screenshot of config object](images/config.png)

### Install project dependency: xhr

We'll use one dependency in our project: [xhr](http://npmjs.org/xhr). It's a great little tool for making http requests in the browser.

Install it with the `npm install` command:

```
npm install --save xhr
```

### Create a link to authenticate with GitHub

Using JavaScript, we're going to add an `a` tag to the html of the page that has an `href` that points to GitHub's authentication url.

Let's make a `renderLink()` function that adds the link to the page:

```js
function renderLink () {
  var url = 'https://github.com/login/oauth/authorize?client_id=' +
  config.client_id + '&scope=user&redirect_uri=' + config.redirect_uri
  var link = document.createElement('a')
  link.href = url
  link.innerHTML = 'Log in with GitHub'
  document.body.appendChild(link)
}
```

Notice how we're using the `config.client_id` and `config.redirect_uri` values from the config to construct the GitHub authentication url.

The full file, including the first line where we require the config file and then the last line where we call the `renderLink()` function should look like this:

```js
var config = require('./config')

function renderLink () {
  var url = 'https://github.com/login/oauth/authorize?client_id=' +
  config.client_id + '&scope=user&redirect_uri=' + config.redirect_uri
  var link = document.createElement('a')
  link.href = url
  link.innerHTML = 'Log in with GitHub'
  document.body.appendChild(link)
}

renderLink()
```

First, let's make sure the github-secret-keeper server is running. If it's still running from earlier when we set it up, great. If not, start it with the `node github-secret-keeper/server.js` command.

Next, run `npm start` if you haven't already, then go to [http://127.0.0.1:9966](http://127.0.0.1:9966/)

You should see a page that looks like this:

![screenshot of github link](images/link.png)

If you click the link, you'll get redirected to GitHub's authorization page:

![screenshot of github authorization page](images/auth.png)

Click the **Authorize application** button and you'll get redirected back to `http://127.0.0.1:9966`, except it will have a `code` query appended to the url.

It will look something like this: `http://127.0.0.1:9966/?code=2091e518204f6fe2dcec`, but the code will be different for each login.

### Make a request with the login code to github-secret-keeper to get an access token

Next we need to make a request to github-secret-keeper that includes that code.

Let's make a `getCode()` function that gets the code from the url:

Require the node.js `querystring` module:

```js
var qs = require('querystring')
```

Then use the `qs.parse()` method in the `getCode()` function:

```js
function getCode () {
  var query = window.location.href.split('?')[1]
  return qs.parse(query).code
}
```

### Get a token from GitHub through github-secret-keeper

Next, we need a `getToken()` function that will take that code as an argument, send the code to github-secret-keeper, and return the token that github-secret-keeper retrieves from GitHub.

First require the xhr module:

```js
var xhr = require('xhr')
```

Then use it in the `getToken()` function:

```js
function getToken (code, callback) {
  var options = {
    url: config.ghAuthHost + '/' + config.client_id + '/' + code,
    json: true
  }

  xhr(options, function (err, res, body) {
    if (err) return callback(err)
    callback(null, body.access_token)
  })
}
```

### Get the user's profile from GitHub with the access token

Now we need a `getProfile()` function that will take that token and make a request for the profile info of the authenticated user:

```js
function getProfile (token, callback) {
  var options = {
    url: 'https://api.github.com/user',
    json: true,
    headers: {
      authorization: 'token ' + token
    }
  }

  xhr(options, function (err, res, body) {
    if (err) return callback(err)
    callback(null, body)
  })
}
```

### Render the user's name with the profile data

Let's make a simple method that renders the name of the user that is authenticated called `renderProfile()`:

```js
function renderProfile (profile) {
  var p = document.createElement('p')
  p.innerHTML = profile.name
  document.body.appendChild(p)
}
```

### Run the app

Finally, let's put it all together with a `start()` command that does the following:

- check the url for the code.
- if the url has a code:
  - get the token
  - get the profile
  - render the profile
- otherwise:
  - render the link to github

Here's how the `start()` function can look:

```js
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

### Full code of index.js

The full code of the index.js file should look like this:

```js
var xhr = require('xhr')
var qs = require('querystring')
var config = require('./config')

start()

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

function renderProfile (profile) {
  var p = document.createElement('p')
  p.innerHTML = profile.name
  document.body.appendChild(p)
}

function getProfile (token, callback) {
  var options = {
    url: 'https://api.github.com/user',
    json: true,
    headers: {
      authorization: 'token ' + token
    }
  }

  xhr(options, function (err, res, body) {
    if (err) return callback(err)
    callback(null, body)
  })
}

function getToken (code, callback) {
  var options = {
    url: config.ghAuthHost + '/' + config.client_id + '/' + code,
    json: true
  }

  xhr(options, function (err, res, body) {
    if (err) return callback(err)
    callback(null, body.access_token)
  })
}

function getCode () {
  var query = window.location.href.split('?')[1]
  return qs.parse(query).code
}

function renderLink () {
  var url = 'https://github.com/login/oauth/authorize?client_id=' +
  config.client_id + '&scope=user&redirect_uri=' + config.redirect_uri
  var link = document.createElement('a')
  link.href = url
  link.innerHTML = 'Log in with GitHub'
  document.body.appendChild(link)
}
```

## Source on GitHub

All the source code for this example is available on GitHub: 

[https://github.com/makerlog/github-login-example](https://github.com/makerlog/github-login-example)

Feel free to make pull requests, fork the code, etc.!


## Next steps

You'll notice that the site will lose the authentication when you refresh the page. Yikes!

An important next step for the project is saving the access token that is returned from github-secret-keeper in a cookie.



# Using cookies with browser-side GitHub auth

Building applications on top of the GitHub API often requires authenticating users, and in this tutorial we'll cover tracking a user's access token with cookies.

## Setting up your project

Navigate to the project directory for the code from the previous tutorial.

To see the code as it was at the end of that first tutorial, check out the post-1 branch of this repository: 

[github.com/makerlog/github-login-example/tree/chapter-1](https://github.com/makerlog/github-login-example/tree/chapter-1)

## Start the github-secret-keeper server

Make sure to remember to start your github-secret-keeper server with this command:

```
node github-secret-keeper/server.js
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
  var token = cookie.get('github-login-example')

  if (token) {
    getProfile(token, function (err, profile) {
      renderProfile(profile)
    })
  } else if (code) {
    getToken(code, function (err, token) {
      cookie.set('github-login-example', token)
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
var token = cookie.get('github-login-example')
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
    cookie.set('github-login-example', token)
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

```js
function renderProfile (profile) {
  var p = document.createElement('p')
  p.innerHTML = profile.name
  document.body.appendChild(p)
}
```

We'll revise the function to add the log out link:

```js
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
    cookie.set('github-login-example', '')
    window.location = window.location
  })
}
```

First we add the log out link to the document body:

```js
var logout = document.createElement('a')
logout.href = '#'
logout.innerHTML = 'log out'
document.body.appendChild(logout)
```

Then we add an event listener to the logout link that resets the cookie and refreshes the page:

```js
logout.addEventListener('click', function (e) {
  e.preventDefault()
  cookie.set('github-login-example', '')
  window.location = window.location
})
```

Go back to the browser and try out the log out button. Success!

## Example source on GitHub

All the source code for this example is available on GitHub: 

[https://github.com/makerlog/github-login-example/tree/chapter-2](https://github.com/makerlog/github-login-example/tree/chapter-2)

Feel free to make pull requests, fork the code, etc.!



# Creating a github-static-auth module

In the third and final chapter to this guide, we'll create a small client module that you can use along with github-secret-keeper to manage github login on static sites.

We'll keep the module simple. It'll have one small set of responsibilities: make API requests to GitHub and github-secret-keeper.

## Project setup

### Create the module folder & files

We'll name this project `github-static-auth`, so navigate to a directory where you'd like to store this module (outside of your example application's directory) and create a new directory named `github-static-auth`:

```
mkdir github-static-auth
cd github-static-auth
```

Inside the github-static-auth directory, create an index.js file:

```
touch index.js
```

To create a package.json file, run `npm init`:

```
npm init
```

Answer the questions it asks. You can press enter for each question to stick with the default answer the command expects.

You should now have a package.json file that looks like this:

```js
{
  "name": "github-static-auth",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

### Dependencies

The only dependency for this module is [xhr](http://npmjs.org/xhr).

Install it with this command:

```
npm install --save xhr
```

In your package.json file xhr will now be listed as a dependency.

## API

To write this module, let's first determine what the usage of this module will be like.

We'll need to require the module, and initialize it with the host of github-secret-keeper and the GitHub client_id of your application.

Then we'll need to send a request to login using the code that's returned by GitHub after visiting their authorization url.

This means that the module will take care of requesting the token through github-secret-keeper, and making a call to GitHub's `user` endpoint to authenticate. Then our module will provide the user's profile object and access token as callback arguments.

### Here's what usage of our module might look like:

```js
/* require the module */
var githubStaticAuth = require('github-static-auth')

/* create the auth object with configuration */
var auth = githubStaticAuth({
  githubSecretKeeper: 'http://127.0.0.1:5000',
  githubClientId: 'your client id'
})

/* log in the user */
auth.login(function (err, profile, token) {
  console.log(profile)
})
```

There might be other ways this module could simplify the authentication process for static sites, but this seems like enough for a solid first version.

Let's write the module code.

## Module code

We'll first walk through each part of the code – some of which will look familiar from previous chapters – and we'll show the full code of the module at the end of this section. The full code of this module will be less than 50 lines.


**Require the `xhr` module:**

```js
var xhr = require('xhr')
```

**Create the function that gets exported using `module.exports`:**

```js
module.exports = function (config) {
  var auth = {}

  auth.login = function auth_login (callback) {
    var code = auth.getCode()
    auth.getToken(code, function (err, token) {
      if (err) return callback(err)
      auth.getProfile(token, callback)
    })
  }

  return auth
}
```

For this module we're creating a simple module that is a function that returns an object with a `login` method. Inside the `login` method we use `getCode`, `getToken`, and `getProfile` methods that are almost exactly the same as the functions we wrote last chapter. We're just making those functions part of this module.

**The `auth.getCode` method:**

```js
auth.getCode = function auth_getCode () {
  var query = window.location.href.split('?')[1]
  return qs.parse(query).code
}
```

**The `auth.getToken` method:**  

```js
auth.getToken = function auth_getToken (code, callback) {
  var options = {
    url: config.githubSecretKeeper + '/' + config.githubClientId + '/' + code,
    json: true
  }

  xhr(options, function (err, res, body) {
    if (err) return callback(err)
    callback(null, body.access_token)
  })
}
```

**The `auth.getProfile` method:**  

```js
auth.getProfile = function auth_getProfile (token, callback) {
  var options = {
    url: 'https://api.github.com/user',
    headers: { authorization: 'token ' + token },
    json: true
  }

  xhr(options, function (err, res, body) {
    if (err) return callback(err)
    callback(null, body, token)
  })
}
```

Note that all these methods should be placed _inside_ the main module function. You'll see how this looks in the full code example below.

### Here's the full code of the module:

```js
var qs = require('querystring')
var xhr = require('xhr')

module.exports = function (config) {
  var auth = {}

  auth.login = function auth_login (callback) {
    var code = auth.getCode()
    auth.getToken(code, function (err, token) {
      if (err) return callback(err)
      auth.getProfile(token, callback)
    })
  }

  auth.getCode = function auth_getCode () {
    var query = window.location.href.split('?')[1]
    return qs.parse(query).code
  }

  auth.getToken = function auth_getToken (code, callback) {
    var options = {
      url: config.githubSecretKeeper + '/' + config.githubClientId + '/' + code,
      json: true
    }

    xhr(options, function (err, res, body) {
      if (err) return callback(err)
      callback(null, body.access_token)
    })
  }

  auth.getProfile = function auth_getProfile (token, callback) {
    var options = {
      url: 'https://api.github.com/user',
      headers: { authorization: 'token ' + token },
      json: true
    }

    xhr(options, function (err, res, body) {
      if (err) return callback(err)
      callback(null, body, token)
    })
  }

  return auth
}
```

Hey, that's less than 50 lines! Small modules are the best for keeping maintenance easy, so it's always exciting when a module can be kept small and simple.

## Refactor application to use github-client-auth

Now that we've created a simple module to handle authentication, we can use it in the example application we're working on!

### Install the dependency

While developing a new module, you can install it as a dependency in other local projects using the `npm link` command. 
Run `npm link` inside the github-static-auth directory:

```
npm link
```

This command creates a symlink so that your module is available globally.

Next, navigate to the directory of your example application and run `npm link github-static-auth`:

```
npm link github-static-auth
```

This command creates a symlink between your node_modules folder and the global symlink of the module that we created earlier.

Using `npm link` allows you to manage local dependencies during development without needing to publish modules to npm and reinstall them everytime you make a change. This is particularly useful when creating new modules that you plan to use as dependencies in an application you're creating.

Next let's refactor that application code to include this new dependency.

### Remove some code

We're going to completely remove the `getCode`, `getToken`, and `getProfile` functions from the example application. 

We're also need to remove the `xhr` and `querystring` dependencies, as those dependencies moved to the `github-static-auth` module.

Do that and the full code should look like this:

```js
var cookie = require('cookie-cutter')
var config = require('./config')

start()

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

function renderLink () {
  var url = 'https://github.com/login/oauth/authorize?client_id=' +
  config.client_id + '&scope=user&redirect_uri=' + config.redirect_uri
  var link = document.createElement('a')
  link.href = url
  link.innerHTML = 'Log in with GitHub'
  document.body.appendChild(link)
}
```

Next we'll add a require statement for `github-static-auth` and revise the `start` function to use our new module.

### Revise some code

Based on the example code we wrote earlier to sketch out what the usage of the github-static-auth module should be like, let's edit the main js file of the example application.

Here's how the first half of the application code should look:

```js
var githubStaticAuth = require('github-static-auth')
var cookie = require('cookie-cutter')
var config = require('./config')

var auth = githubStaticAuth({
  githubSecretKeeper: config.ghAuthHost,
  githubClientId: config.client_id
})

start()

function start () {
  var token = cookie.get('github-auth-example')

  if (token) {
    auth.getProfile(token, function (err, profile) {
      renderProfile(profile)
    })
  } else if (auth.getCode()) {
    auth.login(function (err, profile, token) {
      cookie.set('github-auth-example', token)
      window.location = window.location.origin
    })
  } else {
    renderLink()
  }
}
```

We're adding the `github-static-auth` require statement and revising the `start` function to use the `auth.getProfile`, `auth.getCode`, and `auth.login` methods.

We could simplify this usage even more by moving the cookies code into the module, but it's possible that a developer might want to save their user token in a different way, like a property in a data store or in a browser feature like localstorage. Keeping that token management outside of the module makes it a little easier to integrate into different types of applications.

The full code of your example application should now look like this:

```js
var githubStaticAuth = require('github-static-auth')
var cookie = require('cookie-cutter')
var config = require('./config')

var auth = githubStaticAuth({
  githubSecretKeeper: config.ghAuthHost,
  githubClientId: config.client_id
})

start()

function start () {
  var token = cookie.get('github-auth-example')

  if (token) {
    auth.getProfile(token, function (err, profile) {
      renderProfile(profile)
    })
  } else if (auth.getCode()) {
    auth.login(function (err, profile, token) {
      cookie.set('github-auth-example', token)
      window.location = window.location.origin
    })
  } else {
    renderLink()
  }
}

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

function renderLink () {
  var url = 'https://github.com/login/oauth/authorize?client_id=' +
  config.client_id + '&scope=user&redirect_uri=' + config.redirect_uri
  var link = document.createElement('a')
  link.href = url
  link.innerHTML = 'Log in with GitHub'
  document.body.appendChild(link)
}
```

Hey, now this js file is just barely more than 50 lines. _Excellent._

## That's it!

That concludes our small module development and example application refactoring!

If you run `npm start` in the directory of your example application, you should be able to use it the same as before the refactor.

### You can review the full example code on GitHub here:

[github.com/makerlog/github-login-example](https://github.com/makerlog/github-login-example)

### github-static-auth on npm

I've published the github-static-auth module on npm, so if you're interested, you can use it in other applications you work on. You could even contribute to development of the module! 

Here it is on npm: 

[npmjs.org/github-static-auth](http://npmjs.org/github-static-auth)

And here is the GitHub repository:

[github.com/sethvincent/github-static-auth](https://github.com/sethvincent/github-static-auth)



