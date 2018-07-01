---
id: 198
title: 'toddmotto.com: Is it time to drop jQuery? Essentials to learning JavaScript from a jQuery background'
date: 2013-07-08T09:06:33+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=198
permalink: /2013/07/08/toddmotto-com-is-it-time-to-drop-jquery-essentials-to-learning-javascript-from-a-jquery-background/
dsq_thread_id:
  - "1477347661"
image: /wp-content/uploads/2013/07/JavaScript-logo.png
categories:
  - JavaScript
tags:
  - javascript
  - jquery
---
<strong>Source:</strong> <a href="https://toddmotto.com/is-it-time-to-drop-jquery-essentials-to-learning-javascript-from-a-jquery-background/">https://toddmotto.com/is-it-time-to-drop-jquery-essentials-to-learning-javascript-from-a-jquery-background/</a>

jQuery has been a godsend to pretty much all of us front-end developers since it's release, it's intuitive methods, easy functions make light work of JavaScript's loosely typed language. JavaScript is hard, it's hard to get into, it's much harder than jQuery. But the time is nearly here, going native is going to be the future of front-end - HTML5.

HTML5 doesn't just mean a few extra HTML elements, if you're putting down on your CV/Resume that you know HTML5 because you've used the new elements, then think again! HTML5 covers such a mass of technology, and also alongside it comes ECMAScript 5, the future of JavaScript. Combining HTML5 APIs, of which most require JavaScript, we need to adopt a more native structure of working as each day jQuery becomes less important, and here's why.

This article takes a jQuery lover through some of the harder, more misunderstood, JavaScript methods, functions and more to show how native technology has caught up, how it's not as difficult as it seems and that native JavaScript is probably going to hit you like a brick in the face fairly soon - if it hasn't already. As a front-end developer I am pretty passionate about knowing my tech, and admittedly I began with jQuery and moved to learning JavaScript, I know many others have too. This article is here to talk anyone looking to dive into native JavaScript development over jQuery, and should hopefully open up some doors into the future of your coding.

<!--more-->

<h3>Selectors</h3>

jQuery selectors are the big seller, we don't even have to think about it, selecting our elements is a no brainer, it's super simple. jQuery uses Sizzle, an engine also created by the jQuery Foundation (but available as a standalone) to use as it's selector engine. The mighty code behind Sizzle will make you think twice before overcomplicating your selectors, and the raw JavaScript alternative will make you think twice about jQuery altogether!

<h4>Class selectors</h4>

JavaScript had no native <em>className</em> method for grabbing elements with classes until fairly recent, which I feel has hindered it's popularity from the start. Classes are the best for our HTML/CSS development, but weren't well supported with native JavaScript - makes sense not to want to 'learn JavaScript' and go with jQuery. Until now.

Let's look at the options:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">);</span>

<span class="c1">// JavaScript</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">getElementsByClassName</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>
</code></pre>
</div>

This returns a NodeList. A Node is a JavaScript term for element Object, and a NodeList is an ordered list of Nodes.

<em>Pro tip:</em> the difference between jQuery and native JavaScript when using selectors like these, is that they return a NodeList that you then have to deal with. jQuery takes care of all this for you, covering up what's really happening - but it's really important to know what's happening.

<h4>ID selectors</h4>

The easiest of the pack:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="s1">'#myID'</span><span class="p">);</span>

<span class="c1">// JavaScript</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">getElementById</span><span class="p">(</span><span class="s1">'myID'</span><span class="p">);</span>
</code></pre>
</div>

Returns a single Node.

<h4>Tags</h4>

As easy as the ID selector, the tag name selector returns a NodeList too:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="s1">'div'</span><span class="p">);</span>

<span class="c1">// JavaScript</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">getElementsByTagName</span><span class="p">(</span><span class="s1">'div'</span><span class="p">);</span>
</code></pre>
</div>

<h4>querySelector/querySelectorAll</h4>

This is where things heat up - enter querySelector. Had it not been for jQuery, querySelector may not have made it's way into the JavaScript language as quickly or as efficiently as it has - so we have jQuery to thank for this.

The magic behind querySelector is astounding, it's a multi-purpose native tool that you can use in various instances (this is raw JavaScript). There are two types of querySelector, the first which is plain old <em>document.querySelector('')</em> returns the first Node in the NodeList, regardless of how many Node Objects it might find. The second, ultimately the best and most powerful is <em>document.querySelectorAll('')</em> which returns a NodeList every time. I've been using <em>document.querySelectorAll('')</em> as standard as it's easier to grab the first item in the returned NodeList than it is to reverse engineer <em>document.querySelector('')</em>.

Let's look at some examples, read the comments for better clarification:

<div class="highlight">
<pre><code class="javascript"><span class="cm">/*</span>
<span class="cm"> * Classes</span>
<span class="cm"> */</span>
<span class="c1">// Grab the first .myClass class name</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">);</span>

<span class="c1">// Return a NodeList of all instances of .myClass</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelectorAll</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">);</span>

<span class="cm">/*</span>
<span class="cm"> * ID</span>
<span class="cm"> */</span>
<span class="c1">// Grab the myID id</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'#myID'</span><span class="p">);</span>

<span class="cm">/*</span>
<span class="cm"> * Tags</span>
<span class="cm"> */</span>
<span class="c1">// Return a NodeList of all 'div' instances</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelectorAll</span><span class="p">(</span><span class="s1">'div'</span><span class="p">);</span>
</code></pre>
</div>

querySelectorAll is powerful, and definitely the future. It also supports more complicated selectors like so:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// Grab the last list Node of .someList unordered list</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'ul.someList li:last-child'</span><span class="p">);</span>

<span class="c1">// Grab some data-* attribute</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelectorAll</span><span class="p">(</span><span class="s1">'[data-toggle]'</span><span class="p">);</span>
</code></pre>
</div>

You can also create a smart wrapper function for this, to save typing out <em>document.querySelectorAll('')</em> each time:

<div class="highlight">
<pre><code class="javascript"><span class="kd">var</span> <span class="nx">_</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">(</span> <span class="nx">elem</span> <span class="p">)</span> <span class="p">{</span>
    <span class="k">return</span> <span class="nb">document</span><span class="p">.</span><span class="nx">querySelectorAll</span><span class="p">(</span> <span class="nx">elem</span> <span class="p">);</span>
<span class="p">}</span>
<span class="c1">// Usage</span>
<span class="kd">var</span> <span class="nx">myClass</span> <span class="o">=</span> <span class="nx">_</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">);</span>
</code></pre>
</div>

You could use a <em>$</em> symbol instead of an underscore, totes up to you. It's not ideal to begin a function expression with an underscore, but for demonstration purposes I have.

IE8 supports querySelector CSS2 selectors, I'm not sure why you'd want to perform DOM operations with CSS3 selectors entirely as CSS3 is used for progressive enhancement, whereas functionality can be broken whereas styling isn't nearly as important. If you're doing it right, you're using efficient class names and minimal selectors.

<h3>Class manipulation</h3>

You can extend JavaScript using a prototypal inheritance method, which is what jQuery is doing behind the scenes. HTML5 however is the future, it's growing and legacy browsers are quickly diminishing. It's time to begin using native JavaScript class methods, which again a new feature in HTML5 is the classList - let's do some jQuery comparisons:

<h4>Add class</h4>

Adding a class is easy in jQuery, it does it all for you, taking care of the NodeList array too, we'll come onto this soon.

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="s1">'div'</span><span class="p">).</span><span class="nx">addClass</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>

<span class="c1">// JavaScript</span>
<span class="kd">var</span> <span class="nx">div</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'div'</span><span class="p">);</span>
<span class="nx">div</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>
</code></pre>
</div>

<h4>Remove class</h4>

Same as the above, super simple:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="s1">'div'</span><span class="p">).</span><span class="nx">removeClass</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>

<span class="c1">// JavaScript</span>
<span class="kd">var</span> <span class="nx">div</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'div'</span><span class="p">);</span>
<span class="nx">div</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nx">remove</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>
</code></pre>
</div>

<h4>Toggle class</h4>

Toggle was a really important to the language, often tricky to replicate via <em>prototype</em> methods. Thankfully it's here:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="s1">'div'</span><span class="p">).</span><span class="nx">toggleClass</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>

<span class="c1">// JavaScript</span>
<span class="kd">var</span> <span class="nx">div</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'div'</span><span class="p">);</span>
<span class="nx">div</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nx">toggle</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>
</code></pre>
</div>

<h3>Arrays</h3>

Now we'll push into the more advanced aspects of the JavaScript language, <em>Arrays</em>. Arrays are used to hold values inside one variable, which looks like so:

<div class="highlight">
<pre><code class="javascript"><span class="kd">var</span> <span class="nx">myArray</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'one'</span><span class="p">,</span> <span class="s1">'two'</span><span class="p">,</span> <span class="s1">'three'</span><span class="p">,</span> <span class="s1">'four'</span><span class="p">]</span>
</code></pre>
</div>

jQuery makes this super easy with the <em>$.each();</em> method, which again hides some of the dirty work and makes things easy. JavaScript began with no 'built-in' functionality for iterating over arrays, so we are used to manually working out the items in the array using the <em>length</em> property and iterating over each item incrementally inside a <em>for</em> loop:

<div class="highlight">
<pre><code class="javascript"><span class="kd">var</span> <span class="nx">myArray</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'one'</span><span class="p">,</span> <span class="s1">'two'</span><span class="p">,</span> <span class="s1">'three'</span><span class="p">,</span> <span class="s1">'four'</span><span class="p">]</span>
<span class="k">for</span> <span class="p">(</span><span class="kd">var</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o"><</span> <span class="nx">myArray</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// ...</span>
<span class="p">}</span>
</code></pre>
</div>

Recently, we received an upgrade from this rather manual method to the dedicated <em>forEach</em> method, which is however slower than the above, but does provide callback functionality similar to jQuery's <em>$.each();</em>:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// Bolt the array at the beginning, I like this</span>
<span class="p">[</span><span class="s1">'one'</span><span class="p">,</span> <span class="s1">'two'</span><span class="p">,</span> <span class="s1">'three'</span><span class="p">,</span> <span class="s1">'four'</span><span class="p">].</span><span class="nx">forEach</span><span class="p">(</span><span class="kd">function</span><span class="p">(){</span>
    <span class="c1">// ...</span>
<span class="p">});</span>

<span class="c1">// Or go oldschool with a variable declaration</span>
<span class="kd">var</span> <span class="nx">myArray</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'one'</span><span class="p">,</span> <span class="s1">'two'</span><span class="p">,</span> <span class="s1">'three'</span><span class="p">,</span> <span class="s1">'four'</span><span class="p">];</span>
<span class="nx">myArray</span><span class="p">.</span><span class="nx">forEach</span><span class="p">(</span><span class="kd">function</span><span class="p">(){</span>
    <span class="c1">// ...</span>
<span class="p">});</span>
</code></pre>
</div>

Looking at the jQuery side of things, here's a quick comparison of the two:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="kd">var</span> <span class="nx">myArray</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'one'</span><span class="p">,</span> <span class="s1">'two'</span><span class="p">,</span> <span class="s1">'three'</span><span class="p">,</span> <span class="s1">'four'</span><span class="p">]</span>
<span class="nx">$</span><span class="p">.</span><span class="nx">each</span><span class="p">(</span> <span class="nx">myArray</span><span class="p">,</span> <span class="kd">function</span> <span class="p">(</span> <span class="nx">index</span><span class="p">,</span> <span class="nx">value</span> <span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">value</span><span class="p">);</span>
<span class="p">});</span>

<span class="c1">// JavaScript</span>
<span class="kd">var</span> <span class="nx">myArray</span> <span class="o">=</span> <span class="p">[</span><span class="s1">'one'</span><span class="p">,</span> <span class="s1">'two'</span><span class="p">,</span> <span class="s1">'three'</span><span class="p">,</span> <span class="s1">'four'</span><span class="p">]</span>
<span class="k">for</span> <span class="p">(</span> <span class="kd">var</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o"><</span> <span class="nx">myArray</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span> <span class="p">)</span> <span class="p">{</span>
    <span class="kd">var</span> <span class="nx">value</span> <span class="o">=</span> <span class="nx">myArray</span><span class="p">[</span><span class="nx">i</span><span class="p">];</span>
    <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span> <span class="nx">value</span> <span class="p">);</span>
<span class="p">}</span>
</code></pre>
</div>

<h3>NodeList looping</h3>

A large difference between jQuery is the fact we need to generate a loop using <em>getElementsByClassName</em> or <em>querySelectorAll</em>. For instance, in jQuery, whether one class, or a <em>NodeList</em> exists, the code is identical! This isn't the same with native JavaScript. For instance to add a class in both (notice the difference in the last two native JavaScript methods):

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="kd">var</span> <span class="nx">someElem</span> <span class="o">=</span> <span class="nx">$</span><span class="p">(</span><span class="s1">'.someElem'</span><span class="p">);</span>
<span class="nx">someElem</span><span class="p">.</span><span class="nx">addClass</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>

<span class="c1">// JavaScript - this adds the class to the first Node only!</span>
<span class="kd">var</span> <span class="nx">someElem</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'.someElem'</span><span class="p">);</span>
<span class="nx">someElem</span><span class="p">.</span><span class="nx">classList</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>

<span class="c1">// JavaScript - this adds the class to every Node in the NodeList</span>
<span class="kd">var</span> <span class="nx">someElem</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">querySelectorAll</span><span class="p">(</span><span class="s1">'.someElem'</span><span class="p">);</span>
<span class="k">for</span> <span class="p">(</span><span class="kd">var</span> <span class="nx">i</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span> <span class="nx">i</span> <span class="o"><</span> <span class="nx">someElem</span><span class="p">.</span><span class="nx">length</span><span class="p">;</span> <span class="nx">i</span><span class="o">++</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">someElem</span><span class="p">[</span><span class="nx">i</span><span class="p">].</span><span class="nx">classList</span><span class="p">.</span><span class="nx">add</span><span class="p">(</span><span class="s1">'myClass'</span><span class="p">);</span>
<span class="p">}</span>
</code></pre>
</div>

So what's the difference here? We get a NodeList returned and therefore need to iterate over the NodeList and apply a new class to each. Pretty simple and makes sense. This is the kind of advanced things jQuery takes care of for us. The thing with JavaScript is that it is pretty scary to get started on, but once you're started it's addictive and it's imperative to know what's going on under the hood, as the saying goes.

<h3>Attributes, setting, getting and removing</h3>

JavaScript offers better descriptive, if a little lengthier in character count, methods to dealing with attributes, let's look at the differences.

<h4>Set attributes</h4>

In jQuery, the naming convention isn't as good as native, as the <em>attr();</em> can callback the value as well as set the value, in a way it's clever, but to those learning it could cause confusion. Let's look at how we can set attributes in both:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">).</span><span class="nx">attr</span><span class="p">(</span><span class="s1">'disabled'</span><span class="p">,</span> <span class="kc">true</span><span class="p">);</span>

<span class="c1">// JavaScript</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">).</span><span class="nx">setAttribute</span><span class="p">(</span><span class="s1">'disabled'</span><span class="p">,</span> <span class="kc">true</span><span class="p">);</span>
</code></pre>
</div>

<h4>Remove attributes</h4>

Removing attributes is just as easy:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">).</span><span class="nx">removeAttr</span><span class="p">(</span><span class="s1">'disabled'</span><span class="p">);</span>

<span class="c1">// JavaScript</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">).</span><span class="nx">removeAttribute</span><span class="p">(</span><span class="s1">'disabled'</span><span class="p">);</span>
</code></pre>
</div>

<h4>Get attributes</h4>

This is how we would log the attribute's vale in the Console:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">$</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">).</span><span class="nx">attr</span><span class="p">(</span><span class="s1">'title'</span><span class="p">));</span>

<span class="c1">// JavaScript</span>
<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'.myClass'</span><span class="p">).</span><span class="nx">getAttribute</span><span class="p">(</span><span class="s1">'title'</span><span class="p">));</span>
</code></pre>
</div>

<h4>Data-* attributes</h4>

HTML5 data-* attributes are probably one of the best additions to the HTML specification ever, IMO of course. I use the jQuery <em>.data();</em> API all the time, and also the native JavaScript if it's required:

<div class="highlight">
<pre><code class="html"><span class="nt"><div</span> <span class="na">class=</span><span class="s">"myElem"</span> <span class="na">data-username=</span><span class="s">"Todd"</span><span class="nt">></div></span>

<span class="nt"><script></span>
<span class="c1">// jQuery</span>
<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">$</span><span class="p">(</span><span class="s1">'.myElem'</span><span class="p">).</span><span class="nx">data</span><span class="p">(</span><span class="s1">'username'</span><span class="p">));</span> <span class="c1">// Logs 'Todd'</span>

<span class="c1">// JavaScript - use the getAttribute method, fairly static</span>
<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'.myElem'</span><span class="p">).</span><span class="nx">getAttribute</span><span class="p">(</span><span class="s1">'data-username'</span><span class="p">));</span>
<span class="nt"></script></span>
</code></pre>
</div>

HTML5 introduces the <em>dataset</em> API, which browser support isn't bad, I don't think IE9/10 even support it. For heavy <em>.data();</em> usage, I recommend jQuery as it works in all browsers - even legacy.

<h3>Parsing JSON</h3>

There are neat tricks we can do to parse JSON and create objects too even in plain ol' JavaScript. It's pretty much the same! Let's take an HTML5 custom data-* attribute for a JSON example, grab the attribute, parse the JSON into an Object and then hook into that object:

<div class="highlight">
<pre><code class="html"><span class="nt"><div</span> <span class="na">class=</span><span class="s">"myElem"</span> <span class="na">data-user=</span><span class="s">'{ "name" : "Todd", "id" : "01282183" }'</span><span class="nt">></div></span>

<span class="nt"><script></span>
<span class="c1">// jQuery</span>
<span class="kd">var</span> <span class="nx">myElem</span> <span class="o">=</span> <span class="nx">$</span><span class="p">(</span><span class="s1">'.myElem'</span><span class="p">).</span><span class="nx">data</span><span class="p">(</span><span class="s1">'user'</span><span class="p">);</span> <span class="c1">// Gets the JSON</span>
<span class="kd">var</span> <span class="nx">myJSON</span> <span class="o">=</span> <span class="nx">$</span><span class="p">.</span><span class="nx">parseJSON</span><span class="p">(</span><span class="nx">myElem</span><span class="p">);</span> <span class="c1">// Parses string into JSON Object</span>
<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">myJSON</span><span class="p">.</span><span class="nx">name</span><span class="p">);</span> <span class="c1">// JSON Object, logs 'Todd'</span>
<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">myJSON</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span> <span class="c1">// JSON Object, logs '01282183'</span>

<span class="c1">// JavaScript</span>
<span class="kd">var</span> <span class="nx">myElem</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'.myElem'</span><span class="p">).</span><span class="nx">getAttribute</span><span class="p">(</span><span class="s1">'data-user'</span><span class="p">);</span>
<span class="kd">var</span> <span class="nx">myJSON</span> <span class="o">=</span> <span class="nx">JSON</span><span class="p">.</span><span class="nx">parse</span><span class="p">(</span><span class="nx">myElem</span><span class="p">);</span>
<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">myJSON</span><span class="p">.</span><span class="nx">name</span><span class="p">);</span> <span class="c1">// JSON Object, logs 'Todd'</span>
<span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">myJSON</span><span class="p">.</span><span class="nx">id</span><span class="p">);</span> <span class="c1">// JSON Object, logs '01282183'</span>
<span class="nt"></script></span>
</code></pre>
</div>

<h3>Events</h3>

Events play a massive part in JavaScript, and has had a bad reputation in the past with cross-browser issues. A simple click event in jQuery:

<div class="highlight">
<pre><code class="javascript"><span class="nx">$</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">click</span><span class="p">(</span><span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
    <span class="c1">// ...</span>
<span class="p">});</span>
</code></pre>
</div>

I actually recommend going with jQuery's <em>.on();</em> method should you want to use the click handler:

<div class="highlight">
<pre><code class="javascript"><span class="nx">$</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">on</span><span class="p">(</span><span class="s1">'click'</span><span class="p">,</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
    <span class="c1">// ...</span>
<span class="p">});</span>
</code></pre>
</div>

Two reasons, you can chain the 'on' part like so:

<div class="highlight">
<pre><code class="javascript"><span class="nx">$</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">on</span><span class="p">(</span><span class="s1">'click focus keyup'</span><span class="p">,</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
    <span class="c1">// ...</span>
<span class="p">});</span>
</code></pre>
</div>

This chains (well, binds) a couple of event handlers to register your function with. Any of them will run it. Not to mention you can easily swap them in and out.

Secondly, event delegation with dynamically created JavaScript elements:

<div class="highlight">
<pre><code class="javascript"><span class="nx">$</span><span class="p">(</span><span class="nx">parent</span><span class="p">).</span><span class="nx">on</span><span class="p">(</span><span class="s1">'click'</span><span class="p">,</span> <span class="nx">elem</span><span class="p">,</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{</span>
    <span class="c1">// ...</span>
<span class="p">});</span>
</code></pre>
</div>

This <em>captures</em> the DOM event via a parent event listener. Look up event <em>bubbling</em> and <em>capturing</em> for homework if you're unsure of the difference.

Back to jQuery versus JavaScript now anyway, here's some event handlers:

<div class="highlight">
<pre><code class="javascript"><span class="cm">/*</span>
<span class="cm"> * Click</span>
<span class="cm"> */</span>
<span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">on</span><span class="p">(</span><span class="s1">'click'</span><span class="p">,</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{...});</span>

<span class="c1">// JavaScript</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">onclick</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{...}</span>

<span class="cm">/*</span>
<span class="cm"> * Submit</span>
<span class="cm"> */</span>
<span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">on</span><span class="p">(</span><span class="s1">'submit'</span><span class="p">,</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{...});</span>

<span class="c1">// JavaScript</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">onsubmit</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{...}</span>

<span class="cm">/*</span>
<span class="cm"> * Change</span>
<span class="cm"> */</span>
<span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">on</span><span class="p">(</span><span class="s1">'change'</span><span class="p">,</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{...});</span>

<span class="c1">// JavaScript</span>
<span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">onchange</span> <span class="o">=</span> <span class="kd">function</span> <span class="p">()</span> <span class="p">{...}</span>
</code></pre>
</div>

You see my point...

There is one issue with JavaScript event handlers however, and you can blame Microsoft for this one (again), with their <em>attachEvent</em> handler. Little did they decide to go down their own non-standard route and integrate <em>attachEvent</em> when every other browser was using <em>addEventListener</em>. Still, there is a nice workaround script, provided by John Resig himself which solves this problem for us. addEventListener is very similar to jQuery's chaining of event handler methods, you can attach more than a single handler for each event - it also assists in event bubbling/catching.

<div class="highlight">
<pre><code class="javascript"><span class="nb">document</span><span class="p">.</span><span class="nx">addEventListener</span><span class="p">(</span><span class="s1">'click'</span><span class="p">,</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
    <span class="c1">// ...</span>
<span class="p">},</span> <span class="kc">false</span><span class="p">);</span>
</code></pre>
</div>

<h3>CSS manipulation</h3>

CSS is admittedly nicer in the jQuery object methods, but check out native JavaScript's implementation of this, it's very similar and worth knowing:

<div class="highlight">
<pre><code class="javascript"><span class="c1">// jQuery</span>
<span class="nx">$</span><span class="p">(</span><span class="nx">elem</span><span class="p">).</span><span class="nx">css</span><span class="p">({</span>
    <span class="s2">"background"</span> <span class="o">:</span> <span class="s2">"#F60"</span><span class="p">,</span>
    <span class="s2">"color"</span> <span class="o">:</span> <span class="s2">"#FFF"</span>
<span class="p">});</span>

<span class="c1">// JavaScript</span>
<span class="kd">var</span> <span class="nx">elem</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="nx">elem</span><span class="p">);</span>
<span class="nx">elem</span><span class="p">.</span><span class="nx">style</span><span class="p">.</span><span class="nx">background</span> <span class="o">=</span> <span class="s1">'#F60'</span><span class="p">;</span>
<span class="nx">elem</span><span class="p">.</span><span class="nx">style</span><span class="p">.</span><span class="nx">color</span> <span class="o">=</span> <span class="s1">'#FFF'</span><span class="p">;</span>
</code></pre>
</div>

The above hooks into JavaScript's <em>style</em> object and allows you to set lots of styles with ease.

<h3>Document Ready Function</h3>

jQuery comes built-in with a DOM ready function handler, in which we can safely execute all of our functions knowing the DOM tree is fully populated and any manipulation we do will work and not return <em>undefined</em> (undefined usually means it doesn't exist, or in this case it would).

As we progress towards a future of amazing technology, browsers now fire their own DOM ready function handler, in modern browsers this is called the <em>DOMContentLoaded</em> event and can be fired like so:

<div class="highlight">
<pre><code class="javascript"><span class="nb">document</span><span class="p">.</span><span class="nx">addEventListener</span><span class="p">(</span><span class="s1">'DOMContentLoaded'</span><span class="p">,</span> <span class="kd">function</span><span class="p">()</span> <span class="p">{</span>
    <span class="c1">// DOM ready, run it!</span>
<span class="p">},</span> <span class="kc">false</span><span class="p">);</span>
</code></pre>
</div>

jQuery has had a tendancy to be called <em>the</em> solution and there's no other alternative ever, ever ever. It's bad for upcoming developers to rely on it and it's imperative to learn, or at least have some understanding of, native JavaScript. The more powerful HTML5 becomes, the more we can utilise these rapid HTML5 native capabilities. And the more powerful the features become, the less we need jQuery, the more useless it becomes!

Embrace new technologies now, I'm not suggesting throw away your jQuery workflow and start going native instantly, but a native future is coming - are you ready!