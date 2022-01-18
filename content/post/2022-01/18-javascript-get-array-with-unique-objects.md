---
title: "JavaScript Get Array With Unique Objects"
slug: javascript-get-array-with-unique-objects
date: 2022-01-18T11:38:00+01:00
categories:
 - JavaScript development
tags:
 - array
 - filter
 - javascript
images:
 - /images/javascript-logo.png
---

In JavaScript you can use `Set` to get an array with unique items. However, this does not work with objects. Converting the object to a string before creating the set is the solution.

<!--more-->

Here is an example:

```js
// A list that contains a duplicate object
list = [
  {
    source: "A",
    target: "B",
  },
  {
    source: "B",
    target: "C",
  },
  {
    source: "A",
    target: "B",
  },
];

// Generate set with stringified list objects and then parse to array of objects
unique = [...new Set(list.map((o) => JSON.stringify(o)))].map((s) =>
  JSON.parse(s)
);

// In result there is unique list
console.log(unique);
// [ { source: 'A', target: 'B' }, { source: 'B', target: 'C' } ]
```