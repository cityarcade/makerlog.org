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
The project is built with [virtual-dom]() and [base-element](), two JavaScript modules that I'm still learning how to use! It's almost entirely a browser app. The only server side code is a small server running [gatekeeper](https://github.com/prose/gatekeeper), the same project that powers the oauth authorization for projects like [prose.io](http://prose.io), [geojson.io](http://geojson.io), & [requirebin.com](http://requirebin.com). Those three projects also happen to be the primary inspirations for EditData.org.

## Example usage

Check out this example usage of a tool for generating timelines called Tik Tok and how it can be used with EditData.org: [datanews.github.io/tik-tok/examples/example-editdata.html](http://datanews.github.io/tik-tok/examples/example-editdata.html)

## Contributing to EditData.org
The project is open source, MIT licensed, and there's a [contributing guide](https://github.com/flatsheet/editdata.org/blob/master/CONTRIBUTING.md) and a handful of [open issues](https://github.com/flatsheet/editdata.org/issues). If you're interested in getting involved, it would be exciting to have the help!
