---
title: "Using leaflet-draw with browserify"
slug: using-leaflet-draw-with-browserify
published: true
layout: post
type: post
contributors: [sethvincent]
topics: [javascript, browserify, leaflet]
---

A while back I was working on a simple mapping project and wanted to use [leaflet-draw](https://github.com/Leaflet/Leaflet.draw) with [browserify](http://browserify.org/). I discovered that there was "one weird trick" for getting everything working as expected.

### Install leaflet and leaflet-draw with npm

First, install the modules from npm:

```
npm i --save leaflet leaflet-draw
```


## Create map.js file

### Require the leaflet modules

And here's the trick. You have to require the leaflet-draw module without assigning it to a variable. This adds it to the global scope, and modifies the `L` object to add the drawing-specific methods.

```
var L = require('leaflet');
require('leaflet-draw');
```

For more about Leaflet and browserify, check out this article: [Basics of making maps with leaflet.js & browserify](http://makerlog.org/posts/leaflet-basics/).