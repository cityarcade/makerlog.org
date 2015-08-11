---
layout: post
title: 'Editing data with EditData.org'
contributors: [sethvincent]
topics: [javascript, data, github]
description: "Edit some tabular data & save it to a GitHub gist with this open source tool."
published: true
---

It's so common to need to edit data in a friendly format instead of JSON or CSV. You could use spreadsheet software, but it isn't always well suited to the task, particularly if your data has long text or unusual data types.

I made [EditData.org](http://editdata.org) as a quick & dirty demonstration of a different kind of editor for tabular data. You can import a CSV file or start a new dataset from scratch.

![screenshot of editdata.org](/public/editdata.png)

Right now the app saves your data to GitHub gists, in an upcoming release I'd like to add the ability to edit JSON or CSV files in GitHub repositories.

## The code
The project is built with [virtual-dom](https://github.com/Matt-Esch/virtual-dom) and [base-element](https://github.com/shama/base-element), two JavaScript modules that I'm still learning how to use! It's almost entirely a browser app. The only server side code is a small server running [gatekeeper](https://github.com/prose/gatekeeper), the same project that powers the oauth authorization for projects like [prose.io](http://prose.io), [geojson.io](http://geojson.io), & [requirebin.com](http://requirebin.com). Those three projects also happen to be the primary inspirations for EditData.org.

### Other modules used in the project:
- [base-element](http://npmjs.org/base-element)
- [basscss-grid](http://npmjs.org/basscss-grid)
- [cookie-cutter](http://npmjs.org/cookie-cutter)
- [csskit](http://npmjs.org/csskit)
- [csv-write-stream](http://npmjs.org/csv-write-stream)
- [data-set](http://npmjs.org/data-set)
- [element-class](http://npmjs.org/element-class)
- [envify](http://npmjs.org/envify)
- [extend](http://npmjs.org/extend)
- [from2-array](http://npmjs.org/from2-array)
- [from2-string](http://npmjs.org/from2-string)
- [github-api](http://npmjs.org/github-api)
- [hash-match](http://npmjs.org/hash-match)
- [inherits](http://npmjs.org/inherits)
- [lodash.debounce](http://npmjs.org/lodash.debounce)
- [lodash.union](http://npmjs.org/lodash.union)
- [normalize.css](http://npmjs.org/normalize.css)
- [through2](http://npmjs.org/through2)
- [view-list](http://npmjs.org/view-list)
- [virtual-dom](http://npmjs.org/virtual-dom)
- [wayfarer](http://npmjs.org/wayfarer)
- [xhr](http://npmjs.org/xhr)

## Example usage

Check out this example usage of a tool for generating timelines called Tik Tok and how it can be used with EditData.org: [datanews.github.io/tik-tok/examples/example-editdata.html](http://datanews.github.io/tik-tok/examples/example-editdata.html)

## Contributing to EditData.org
The project is open source, MIT licensed, [the code is on GitHub](https://github.com/flatsheet/editdata.org), and there's a [contributing guide](https://github.com/flatsheet/editdata.org/blob/master/CONTRIBUTING.md) and a handful of [open issues](https://github.com/flatsheet/editdata.org/issues). If you're interested in getting involved, it would be exciting to have the help!
