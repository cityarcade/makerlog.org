---
title: 'A client module for working with the GitHub authorization API and github-secret-keeper'
contributors: [sethvincent]
topics: [javascript, authentication, github, github-auth]
description: "Make it easier to authenticate with GitHub on static sites."
published: true
---

Building applications on top of the GitHub API is fun. GitHub's API can take care of users & permissions as well as integrate your app with the git repositories of your users.

And you can do this with mostly JavaScript in the browser. The only part of the GitHub API that requires a POST request to be sent from a server is for the initial authorization of the user. Luckily there's an open source project called [github-secret-keeper](https://github.com/HenrikJoreteg/github-secret-keeper) that serves that single purpose of authenticating users, and you can set it up once and use it with multiple applications.

To make the browser-side code that works with github-secret-keeper and the GitHub API a little simpler, I created [github-static-auth](https://github.com/sethvincent/github-static-auth), a module that takes care of the HTTP requests to github-secret-keeper and endpoint of the GitHub API that authenticates a user.

## Getting started

Install the module into your project using npm:

```
npm i --save github-static-auth
```

This module works with bundlers like browserify and webpack.

### Setting up github-secret-keeper

### Using github-static-auth in your app

## More info

### Github repos:
- [github.com/sethvincent/github-static-auth](https://github.com/sethvincent/github-static-auth)
- [github.com/HenrikJoreteg/github-secret-keeper](https://github.com/HenrikJoreteg/github-secret-keeper)
