---
title: 'JS snippet: Set tallest height on siblings'
date: 2017-08-06T20:40:15+00:00
author: Janik von Rotz
permalink: /2017/08/06/js-snippet-set-tallest-height-on-siblings/
categories:
  - JavaScript
tags:
  - function
  - height
  - js
  - no-jquery
  - pure
  - set
  - snippet
  - tallest
  - vanilla
---
This post is another contribution to "I hope that I never have to use jQuery again". The problem solved this time is quite simple. We want to set the same height for a group of divs. So not like this:

![Untitled](/wp-content/uploads/2017/07/Not-equal-heights-on-divs.png)

<!--more-->

If you're not using any js library (other than jQuery of course) add this `forEach` function declaration to your js file.

```js
var forEach = function (array, callback, scope) {
  for (var i = 0; i < array.length; i++) {
    callback.call(scope, i, array[i]); 
  }
}
```

Now the snippet below solves our problem. It looks for the tallest divs and set its height to its siblings.

```js
var collection = document.getElementsByClassName('collection')[0]
if(collection) {
    var boxes = collection.getElementsByClassName('box')
    var maxHeight = 0

    // get max height
    forEach(boxes, function (index, box) {
        if (box.offsetHeight >= maxHeight) {
            maxHeight = box.offsetHeight
        }
    })

    // set max height
    forEach(boxes, function (index, box) {
        box.style.height = `${maxHeight}px`
    })
}
```

Simply set the class name of the parent element holding the divs and the class name of the divs.

Quite easy, isn't it?