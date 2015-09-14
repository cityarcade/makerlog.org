---
title: 'Using gatekeeper to authenticate to GitHub with browser-side JavaScript'
contributors: [sethvincent]
topics: [javascript, authentication, github]
description: "Allow GitHub login on your static sites."
published: true
---

[Gatekeeper](https://github.com/prose/gatekeeper) is a node.js project for allowing users to login to GitHub from static sites. To complete the authentication, GitHub requires the `client_secret` of the API keys to remain secret, so it can't be sent with browser-side JavaScript. 

That's what gatekeeper is for. It's a simple server that has one job: accept requests with a GitHub authentication code, send the code & API keys to GitHub, and send back a response to your site with the access token for that user.

This article will walk you through setting up gatekeeper and using it to get the json object of the profile for users that log in to GitHub through your site.

## Prerequisites

You'll need [node.js](http://nodejs.org/download) and [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) installed on your computer before starting this tutorial, and you'll also need an account on [github.com](http://github.com).

## Setting up your project

Let's do some initial setup, including creating a new folder, .gitignore file, a config.js file a package.json file, and an index.js file for your browser code.

We'll also create an application on GitHub to get the API keys we'll need to make requests to the GitHub API. Let's start with that.

### Create an application on GitHub

If you haven't already, create an account on [github.com](http://github.com) and log in.

Click on your name at the top right of the screen, then click **Setting**.

![screenshot of settings link](/public/posts/github-auth/settings.png)

Then click the **Applications** link in the menu on the left.

![screenshot of applications link](/public/posts/github-auth/applications.png)

Next click **Developer Application**, and then **Register new application**.

![screenshot of register new application button](/public/posts/github-auth/register.png)

Finally, fill out the **Application name**, **Homepage URL**, and **Authorization callback URL** for your application.

![screenshot of new application form](/public/posts/github-auth/form.png)

You can use these settings for your new application:

- Application name: `github-auth-dev` (or whatever you want to call it)
- Homepage URL: `http://127.0.0.1:9966`
- Authorization callback URL: `http://127.0.0.1:9966`

Click the **Register application** button and you're finished registering the application!

On the next screen you'll see the **Client ID** and **Client Secret** that we'll need later in the tutorial. Keep this browser tab open for later.

### Create the project folder

To create the folder, open a terminal window, navigate to a directory that you use to store projects, and run this command:

```
mkdir github-auth
```

You can replace `github-auth` with whatever you want to call your project.

Next, change directory into the project using the `cd` command:

```
cd github-auth
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
gatekeeper
```

**Tips for using `nano`:** To save a file, press the keys `Control` and `O` at the same time, then press `Enter`. To exit nano, press the keys `Control` and `X` at the same time.

The .gitignore file now excludes the error log `npm-debug.log` that npm creates when an error occurs, the `node_modules` folder where all dependencies are stored, and the `gatekeeper` folder. I like installing gatekeeper inside of the project directory for simplicity, but we don't want to check the gatekeeper code into our project, so we exclude it with .gitignore.

### Create the config.js file

The config.js file is where we'll store global variables like the url of the gatekeeper server, the redirect uri that we need to send to GitHub, and the `client_id` part of the GitHub API keys of the application.

Using `nano`, create the config.js file:

```
nano config.js
```

You'll find that you'll eventually need two sets of this configuration: one set for development, and one set for production. To prepare for this, let's set up the config.js file like so:

```
var config = {
  development: {},
  production: {}
}

module.exports = config[process.env.NODE_ENV]
```

For now we'll just fill in the `development` object. Later when you deploy your project you'll want to create a second application on GitHub and fill in the `production` object with the appropriate values.

Here's the config.js file updated with the `development` config:

```
var config = {
  development: {
    client_id: 'your client id',
    redirect_uri: 'http://127.0.0.1:9966',
    gatekeeper: 'http://127.0.0.1:9999'
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

## Installing gatekeeper

To install gatekeeper we'll complete a few steps: cloning the repository, installing gatekeeper's dependencies, and setting up the config for gatekeeper.

### Cloning the repository

Clone the gatekeeper repository with the following command:

```
git clone https://github.com/prose/gatekeeper.git
```

This will create a `gatekeeper` folder in your project directory.

Change directory into gatekeeper with the `cd` command:

```
cd gatekeeper
```

### Install gatekeeper's dependencies

Use the `npm install` command to install all of gatekeeper's dependencies:

```
npm install
```

### Edit gatekeeper's config.json file

Gatekeeper also has a config file, but this one is called **config.json**.

Use `nano` to edit the file.

```
nano config.json
```

You should see contents like this:

```
{
  "oauth_client_id": "GITHUB_APPLICATION_CLIENT_ID",
  "oauth_client_secret": "GITHUB_APPLICATION_CLIENT_SECRET",
  "oauth_host": "github.com",
  "oauth_port": 443,
  "oauth_path": "/login/oauth/access_token",
  "oauth_method": "POST"
}
```

Change the `oauth_client_id` and `oauth_client_secret` to the **Client ID** and **Client Secret** from the GitHub application you created earlier. That should be the only values you'll need to change in the config.json file.

### Start the gatekeeper server

You can start the gatekeeper server with this command:

```
node server.js
```

If you are in the root directory of your project instead of inside the gatekeeper directory, you can start gatekeeper with a slight revision:

```
node gatekeeper/server.js
```

After starting the server you should see output similar to this:

![screenshot of gatekeeper server output](/public/posts/github-auth/server.png)

## Start your app!

Alright, we've got gatekeeper all set up. Now we can get to the fun part: authenticating users.

In this section we'll install project & development dependencies, set up some npm scripts, and write some JavaScript for making requests to gatekeeper and GitHub.

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

```
{
  "name": "github-auth",
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

```
"scripts": {
  "start": "budo index.js:bundle.js --live -- -t [ envify --NODE_ENV development ]"
},
```

The `--live` part of the command makes it so budo watches for changes, and when you save a file, re-bundles the JavaScript and restarts your browser window.

The `-t [ envify --NODE_ENV development ]` part of the command is what tells browserify to make `process.env.NODE_ENV` equal the value `development`.

### npm script: `npm run bundle`

Next let's add a `bundle` script. This is what you will use to bundle your code for production.

The `bundle` script should look like this:

```
"bundle": "browserify index.js -t [ envify --NODE_ENV production ] -o  assets/bundle.js"
```

Note that now the `[ envify --NODE_ENV production ]` section of the command will make `process.env.NODE_ENV` equal the value `production`.

The `scripts` object should now look something like this:

```
"scripts": {
  "bundle": "browserify index.js -t [ envify --NODE_ENV production ] -o  assets/bundle.js",
  "start": "budo index.js:bundle.js --live -- -t [ envify --NODE_ENV development ]"
},
```

Let's make sure that the `npm start` script is working.

Open up the index.js file and add a `console.log` statement that logs the `config` object.

Like this:

```
var config = require('./config')

console.log(config)
```

Next, run `npm start`.

You should see output similar to this:

```
> github-auth@1.0.0 start /Users/sethvincent/workspace/makerlog/examples/github-auth
> budo index.js:bundle.js -- -t [ envify --NODE_ENV development ]

{"time":"2015-08-25T01:20:16.285Z","hostname":"pizza-2.local","pid":49491,"level":"info","name":"budo","message":"Server running at http://localhost:9966/","type":"connect","url":"http://localhost:9966/"}
{"time":"2015-08-25T01:20:25.489Z","hostname":"pizza-2.local","pid":49491,"level":"info","name":"budo","url":"/","type":"generated"}
{"time":"2015-08-25T01:20:25.699Z","hostname":"pizza-2.local","pid":49491,"level":"info","name":"budo","url":"/bundle.js","type":"static"}
```

Now go to [http://localhost:9966](http://localhost:9966) in your browser and open up the JavaScript console in your browser's developer tools. You should see the config object logged like this:

![screenshot of config object](/public/posts/github-auth/config.png)

### Install project dependency: xhr

We'll use one dependency in our project: [xhr](http://npmjs.org/xhr). It's a great little tool for making http requests in the browser.

Install it with the `npm install` command:

```
npm install xhr
```

### Create a link to authenticate with GitHub

Using JavaScript, we're going to add an `a` tag to the html of the page that has an `href` that points to GitHub's authentication url.

Let's make a `renderLink()` function that adds the link to the page:

```
function renderLink () {
  var url = 'https://github.com/login/oauth/authorize?client_id=' + config.client_id + '&scope=user&redirect_uri=' + config.redirect_uri
  var link = document.createElement('a')
  link.href = url
  link.innerHTML = 'Log in with GitHub'
  document.body.appendChild(link)
}
```

Notice how we're using the `config.client_id` and `config.redirect_uri` values from the config to construct the GitHub authentication url.

The full file, including the first line where we require the config file and then the last line where we call the `renderLink()` function should look like this:

```
var config = require('./config')

function renderLink () {
  var url = 'https://github.com/login/oauth/authorize?client_id=' + config.client_id + '&scope=user&redirect_uri=' + config.redirect_uri
  var link = document.createElement('a')
  link.href = url
  link.innerHTML = 'Log in with GitHub'
  document.body.appendChild(link)
}

renderLink()
```

First, let's make sure the gatekeeper server is running. If it's still running from earlier when we set it up, great. If not, start it with the `node gatekeeper/server.js` command.

Next, run `npm start` if you haven't already, then go to [http://127.0.0.1:9966](http://127.0.0.1:9966/)

You should see a page that looks like this:

![screenshot of config object](/public/posts/github-auth/link.png)

If you click the link, you'll get redirected to GitHub's authorization page:

![screenshot of config object](/public/posts/github-auth/auth.png)

Click the **Authorize application** button and you'll get redirected back to `http://127.0.0.1:9966`, except it will have a `code` query appended to the url.

It will look something like this: `http://127.0.0.1:9966/?code=2091e518204f6fe2dcec`, but the code will be different for each login.

### Make a request with the login code to gatekeeper to get an access token

Next we need to make a request to gatekeeper that includes that code.

Let's make a `getCode()` function that gets the code from the url:

Require the node.js `querystring` module:

```
var qs = require('querystring')
```

Then use the `qs.parse()` method in the `getCode()` function:

```
function getCode () {
  var query = window.location.href.split('?')[1]
  return qs.parse(query).code
}
```

### Get a token from GitHub through gatekeeper

Next, we need a `getToken()` function that will take that code as an argument, send the code to gatekeeper, and return the token that gatekeeper retrieves from GitHub.

First require the xhr module:

```
var xhr = require('xhr')
```

Then use it in the `getToken()` function:

```
function getToken (code, callback) {
  var options = {
    url: config.gatekeeper + '/authenticate/' + code,
    json: true
  }

  xhr(options, function (err, res, body) {
    if (err) return callback(err)
    callback(null, body.token)
  })
}
```

### Get the user's profile from GitHub with the access token

Now we need a `getProfile()` function that will take that token and make a request for the profile info of the authenticated user:

```
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

```
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

### Full code of index.js

The full code of the index.js file should look like this:

```
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
    url: config.gatekeeper + '/authenticate/' + code,
    json: true
  }

  xhr(options, function (err, res, body) {
    if (err) return callback(err)
    callback(null, body.token)
  })
}

function getCode () {
  var query = window.location.href.split('?')[1]
  return qs.parse(query).code
}

function renderLink () {
  var ghURL = 'https://github.com/login/oauth/authorize?client_id=' + config.client_id + '&scope=user&redirect_uri=' + config.redirect_uri
  var link = document.createElement('a')
  link.href = ghURL
  link.innerHTML = 'Log in with GitHub'
  document.body.appendChild(link)
}
```

## Next steps

You'll notice that the site will lose the authentication when you refresh the page. Yikes!

An important next step for the project is saving the access token that is returned from gatekeeper in a cookie, localstorage, or indexeddb.

Watch for a followup post about that! If you want to get emailed when the next post is released sign up for our email list:

{% include newsletter.html %}


## Source on GitHub

All the source code for this example is available on GitHub: [https://github.com/sethvincent/github-auth-example](https://github.com/sethvincent/github-auth-example)

Feel free to make pull requests, fork the code, etc.!

Gatekeeper was originally developed for [prose.io](http://prose.io) and you'll find it in the prose GitHub organization: [https://github.com/prose/gatekeeper](https://github.com/prose/gatekeeper)

## Projects that use gatekeeper:
- [prose.io](https://github.com/prose/prose) – edit prose files that are on github
- [requirebin.com](https://github.com/maxogden/requirebin) – a javascript bin that let's you require modules from npm
- [geojson.io](https://github.com/mapbox/geojson.io) – a tool for editing geojson
- [editdata.org](https://github.com/flatsheet/editdata.org) – a tool for editing tabular data and saving it to github

## Next post in the series: adding cookies

Take the next steps with this tutorial: [Using cookies with browser-side GitHub auth](/posts/using-cookies-with-browser-side-github-auth).

_Have a project that uses gatekeeper and want it listed? [Send a pull request](https://github.com/civicmakerlab/makerlog.org/blob/master/_posts/2015-08-24-gatekeeper-for-authenticating-with-github.md)_

## This post is open source
Spot a typo? Have a great addition to make? [Send a pull request](https://github.com/civicmakerlab/makerlog.org/blob/master/_posts/2015-08-24-gatekeeper-for-authenticating-with-github.md)