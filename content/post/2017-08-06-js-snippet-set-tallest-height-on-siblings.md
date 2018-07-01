---
id: 4375
title: 'JS snippet: Set tallest height on siblings'
date: 2017-08-06T20:40:15+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4375
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

<img src="https://janikvonrotz.ch/wp-content/uploads/2017/07/Not-equal-heights-on-divs.png" alt="" width="381" height="321" class="aligncenter size-full wp-image-4389" />

<!--more-->

If you're not using any js library (other than jQuery of course) add this `forEach` function declaration to your js file.

[code lang="js"]
var forEach = function (array, callback, scope) {
  for (var i = 0; i &lt; array.length; i++) {
    callback.call(scope, i, array[i]); 
  }
}
[/code]

Now the snippet below solves our problem. It looks for the tallest divs and set its height to its siblings.

[code lang="js"]
var collection = document.getElementsByClassName('collection')[0]
if(collection) {
    var boxes = collection.getElementsByClassName('box')
    var maxHeight = 0

    // get max height
    forEach(boxes, function (index, box) {
        if (box.offsetHeight &gt;= maxHeight) {
            maxHeight = box.offsetHeight
        }
    })

    // set max height
    forEach(boxes, function (index, box) {
        box.style.height = `${maxHeight}px`
    })
}
[/code]

Simply set the class name of the parent element holding the divs and the class name of the divs.

Quite easy, isn't it?