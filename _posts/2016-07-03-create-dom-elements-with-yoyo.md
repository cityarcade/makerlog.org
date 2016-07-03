---
title: "Introduction to creating DOM elements with yo-yo.js"
description: "A simple JS API for creating reusable elements that can be diffed and updated similar to React and other virtual dom modules – without the virtual part."
topics: [javascript, dom, tcby, yo-yo, choo, bel]
contributors: [sethvincent]
published: true
---

## What is this
[yo-yo](https://npmjs.com/yo-yo) is a cute name for a module that creates and updates DOM elements. You can use it in place of modules like [react](https://npmjs.com/react) or [virtual-dom](https://npmjs.com/virtual-dom).

### Some features of yo-yo:
- Use yo-yo in the browser or on the server
- Instead of using a virtual DOM like react, yo-yo uses the actual DOM to diff changes and make efficient updates
- No need for JSX. Instead yo-yo uses tagged template literals, a built-in feature of JavaScript
- Small file size (4kb minified and gzipped)
- No huge dependencies or devDependencies
- You can support older browsers using the [yo-yoify browserify transform](https://npmjs.com/yo-yoify)

## How to use yo-yo

Here are a few yo-yo usage examples to get you started.

### Follow along:
- create a directory for this example code
- install yo-yo with npm: `npm install yo-yo`
- create a file named `index.js` to type in examples
- install budo, a development server for browserify: `npm install -g budo`

### Creating elements

To create an element, `require` the `yo-yo` module, then put html inside of the backticks after `yo`:

```js
var yo = require('yo-yo')

var element = yo`
  <p>hi</p>
`
```

### Attaching elements to the DOM

There's nothing fancy about attaching an element made with yo-yo to the DOM. Use `appendChild` to attach the element to the `body` or any other existing html:

```js
var yo = require('yo-yo')

var element = yo`
  <p>hi</p>
`

document.body.appendChild(element)
```

### Using variables

A nice part about template literals is the ability to interpolate values into the string:

```js
var yo = require('yo-yo')

var message = 'oh cool'

var element = yo`
  <p>${message}</p>
`

document.body.appendChild(element)
```

### Updating elements

To update an element we use the `yo.update` method. The first argument to `yo.update` is the existing element you want to update. The second argument is the element with the new changes that you want to add to the DOM. In this example we increase the `count` variable every second:

```js
var yo = require('yo-yo')

var count = 0

var element = yo`
  <p>count: ${count}</p>
`

setInterval(function () {
  count++
  var newElement = yo`
    <p>count: ${count}</p>
  `
  yo.update(element, newElement)
}, 1000)

document.body.appendChild(element)
```

### Updating elements in response to user input

We can use an `onclick` handler on a `<button>` element to update the count rather than updating every second:

```js
var yo = require('yo-yo')

var count = 0

var element = yo`
  <p>count: ${count}</p>
`

var input = yo`
  <button onclick=${onclick}>increase count</button>
`

function onclick () {
  count++
  var newElement = yo`
    <p>count: ${count}</p>
  `
  yo.update(element, newElement)
}

document.body.appendChild(element)
document.body.appendChild(input)
```

### Simplifying multiple elements with a render function

The above example is a little messy with the duplicate `document.body.appendChild` statements and multiple elements created with `yo`.

Let's clean up the example by using a `render` function that returns the DOM node created by `yo`. Note that when we have multiple elements we have to wrap it in a parent element so that there is only one root element:

```js
var yo = require('yo-yo')

var count = 0

function render () {
  return yo`
    <div id="app">
      <p>count: ${count}</p>
      <button onclick=${onclick}>increase count</button>
    </div>
  `
}

var app = render()

function onclick () {
  count++
  var update = render()
  yo.update(app, update)
}

document.body.appendChild(app)
```

### Supporting older browsers

It's recommended to use [browserify](https://npmjs.com/browserify) to bundle JS dependencies when using yo-yo. Additionally you can use the [yo-yoify](https://npmjs.com/yo-yoify) transform for browserify to convert tagged template literals into plain DOM-related JavaScript.

#### Example of setting up browserify & yo-yoify

Install as devDependencies:

```
npm install --save-dev browserify yo-yoify
```

Create a `bundle` npm script in your package.json file:

```
"scripts": {
  "browserify index.js -t yo-yoify -p bundle.js"
}
```

## Related projects

### choo

[choo](https://npmjs.com/choo) is a pleasant little framework based on yo-yo and a handful of other small modules. Check it out if you're looking for a higher-level approach, or to get a sense of how yo-yo might fit into a larger project.

### bel & morphdom

yo-yo's dependencies are [bel](https://npmjs.com/bel) and [morphdom](https://npmjs.com/morphdom). bel in turn uses [hyperx](https://npmjs.com/hyperx) to create DOM nodes from the tagged template literals, and morpddom provides the DOM diffing functionality. Try them out if you're looking for a lower-level approach to tagged template strings and DOM diffing, or want to learn more about how yo-yo works.
