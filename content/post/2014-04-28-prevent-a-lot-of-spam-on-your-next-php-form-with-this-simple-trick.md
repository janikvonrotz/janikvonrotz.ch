---
id: 2121
title: Prevent a lot of spam on your next php form with this simple trick
date: 2014-04-28T12:08:19+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2121
permalink: /2014/04/28/prevent-a-lot-of-spam-on-your-next-php-form-with-this-simple-trick/
dsq_thread_id:
  - "2644724256"
image: /wp-content/uploads/2014/04/Stop-Spam.jpg
categories:
  - IT Security
  - PHP
tags:
  - css
  - field
  - form
  - hidden
  - php
  - prevent
  - robot
  - security
  - spam
  - stop
---
Spam bots were parsing websites html to code and searching for form patterns. What they luckily don't do in most cases is running javascript or applying css code.
This behaviour is a good way to tell a human from a spambot apart.

Here is a simple example of how to make use of this behaviour to prevent a lot of spam.
<!--more-->
[code lang="php"]
&lt;?php //post the form fields from below
    $name = $_POST['name'];
    $machine = $_POST['machine'];
    if ($machine != &quot;&quot;)
    {
        exit(); //if a spambot filled out the &quot;machine&quot;
                //field, we don't proceed
    }
    else
    {
        //validate the name and do stuff with it
    }
?&gt;
 
&lt;!DOCTYPE html&gt;
&lt;html&gt;
    &lt;head&gt;
        &lt;title&gt;Test Form&lt;/title&gt;
        &lt;style&gt;
            /* hide the &quot;machine&quot; field */
            .machine { display: none; }
        &lt;/style&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;form method=&quot;post&quot; action=&quot;&quot;&gt;
            &lt;input name=&quot;name&quot; /&gt;
            &lt;!-- below field is hidden with css --&gt;
            &lt;input name=&quot;machine&quot; class=&quot;machine&quot; /&gt;
            &lt;!-- edit - show a warning (also hidden) to users with CSS disabled --&gt;
            &lt;label for=&quot;machine&quot; class=&quot;machine&quot;&gt;If you are a human, don't fill out this field!&lt;/label&gt;
            &lt;input type=&quot;submit&quot; /&gt;
        &lt;/form&gt;
    &lt;/body&gt;
&lt;/html&gt;
[/code]

The idea here is that spambots will fill out all fields in the form. So we hide one of the fields with CSS (so users don't see it) and if it's filled out, we don't allow the submission to complete.

Latest version of this snippet: [https://gist.github.com/11363197](https://gist.github.com/11363197)