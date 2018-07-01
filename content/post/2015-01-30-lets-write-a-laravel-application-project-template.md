---
id: 2902
title: Let’s write a Laravel application – Project template
date: 2015-01-30T10:28:03+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2902
permalink: /2015/01/30/lets-write-a-laravel-application-project-template/
dsq_thread_id:
  - "3469034417"
image: /wp-content/uploads/2015/01/laravel-logo-e1422466263489.png
categories:
  - Laravel
tags:
  - artisan
  - auto
  - automatically
  - bootstrap
  - bower
  - css
  - github
  - grunt
  - javascript
  - laravel
  - live
  - livereload
  - minify
  - nodejs
  - npm
  - php
  - project
  - refresh
  - reload
  - task
  - template
  - uglify
  - update
---
In web development there are tons of programs and tools and due to that also complex and very different development strategies.
Luckily dependency handling got a lot easier. For my Laravel project setup we will use 3 different package managers.
Every package manager of course manages a different resource, we will use composer for php packages, npm for everything related to Node.js and Bower for web packages.
<!--more-->
<img src="https://janikvonrotz.ch/wp-content/uploads/2015/01/Web-Technologies-1024x766.png" alt="Web Technologies" width="720" height="539" class="alignnone size-large wp-image-2903" />

By following the instructions of this guide you'll get a very advanced Laravel project template with the following features:

* Live Reloading of your browser.
* Twitter Bootstrap included.
* Automatic minification and bundling of CSS and JavaScript files.
* The most recent web technologies at your hand.
* A default blade template ready to run.

# Install

**1. Bitnami Nginx Webstack**

First of all we need a php executable and a MySQL server. The most convienient way to deploy these services is by installing a predefined stack.
Actually we don't need a webserver, however if you're using phpmyadmin it's already onboard.

[Get Bitnami Nginx Webstack](https://bitnami.com/stack/nginx)

**2. Composer**

As I've said composer is a php package manager. As Laravel is a composer package we need this tool to deploy the Laravel project. You can get composer [here]().

[Get Composer](https://getcomposer.org/)

**3. Node.js**

The npm package manager, which will provide us Grunt and Bower is part of Node.js.

[Get Node.js](http://nodejs.org/)

**4. Atom.io**

As a web developer you might have heard of Sublime, well suck that! Atom.io is out of question way better. It's opensource, it's customizable down to the core and runs on the most promising technologies. If you're using Sublime and reading this, the time has come to flush your workflow and get startet with Atom.io.

[Get Atom.io](https://atom.io/)

# Configure

We assume you have installed all the tools above properly and ready to run.

**1. Install Laravel**

First download the Laravel package with composer. It doesn't matter where you'll do this. This package will globally available on your host.

	composer global require "laravel/installer=~1.1"

Navigate to a directory with your command line where you want to install your first Laravel project.

	composer create-project laravel/laravel <project name> --prefer-dist

**2. Install Bower and Bootstrap**

Navigate into your project directory.

	cd <project name>

Create a npm configuration file.

	npm init

Then install Bower with npm.

	npm install -g bower

Create a Bower configuration file.

	bower init

And install Bootstrap and jQuery.

	bower install bootstrap jquery --save-dev

**3. Install and Configure Grunt**

Install the Grunt command line tool with npm.

	npm install -g grunt-cli

Then install the Grunt packages.

	npm install grunt --save-dev

Now add the additional Grunt packages to your npm config file `package.json`.

[code]
&quot;devDependencies&quot;: {
    &quot;grunt&quot;: &quot;~0.4.5&quot;,
    &quot;grunt-contrib-copy&quot;: &quot;~0.7.0&quot;,
    &quot;grunt-contrib-cssmin&quot;: &quot;~0.11.0&quot;,
    &quot;grunt-contrib-uglify&quot;: &quot;~0.7.0&quot;,
    &quot;grunt-contrib-watch&quot;: &quot;~0.6.1&quot;,
    &quot;grunt-bg-shell&quot;: &quot;~2.3.1&quot;
}
[/code]

If you did so run the npm installer.

	npm install

Create a Grunt task file `Gruntfile.js` in the root directory of your project.

And add this content.

[code]
module.exports = function(grunt){

  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),

    cssmin: {
      files: {
        src: [
          'public/style.css',
          'bower_components/bootstrap/dist/css/bootstrap.css'
        ],
        dest: 'public/all.min.css'
      }
    },

    uglify: {
      files: {
        src: [
          'public/app.js',
          'bower_components/jquery/dist/jquery.js',
          'bower_components/bootstrap/dist/js/bootstrap.js'
        ],
        dest:  'public/all.min.js'
      }
    },

    bgShell: {
      _defaults: {
        bg: true
      },
      runLaravel: {
        cmd: 'php artisan serve'
      }
    },

    watch:{
      css:{
        files: [
          '/public/style.css'
        ],
        tasks: ['cssmin']
      },
      js: {
        files: [
          '/public/app.js'
        ],
        tasks: ['uglify']
      },
      livereload: {
        options: {
            livereload: true
        },
        files: [
            'app/views/**/*.php',
            'public/*.css',
            'public/*.js'
        ]
      }
    }
  });

  grunt.loadNpmTasks('grunt-contrib-copy');
  grunt.loadNpmTasks('grunt-contrib-cssmin');
  grunt.loadNpmTasks('grunt-contrib-watch');
  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-bg-shell');

  grunt.registerTask('default', ['cssmin','uglify','bgShell:runLaravel','watch']);

};
[/code]

This file provides the following features:

* Bundling and minifcation of your Bootstrap and custom CSS files and Bootstrap, jQuery and custom JavaScript files.
* It will start Laravel webserver.
* Provide the Live Reload feature.

As always you can extend the Grund configuration according to your requirements. Some my want to recompile the Bootstrap less files with an updated fonts folder or add php unitiy testing.

**4. Configure Laravel**

In order to make use of the Live Reload feature, you can either add this JavaScript to your template.

	<script src="//localhost:35729/livereload.js"></script>

Or (what I recommend) download and install following blade template files.

[Download template files](https://janikvonrotz.ch/wp-content/uploads/2015/01/Laravel-default-template.zip) or get the latest version [here](https://gist.github.com/janikvonrotz/68f4da6bc6a4374d9f9b).

Add these files to a new folder called default in the app views folder `<project name>\app\views\default`

Then delete the file `<project name\app\views\home.php` and add a new file `<project name>\app\views\home.blade.php` with the following content.

[code]
@extends('default.master')
@section('content')
Content goes here
@stop
[/code]

This template assumes that you distinct between a local and a productive environment. You have to update two configuration files before you'll run your project.

In the file `<project name>\boostrap\start.php` add your hostname to the detect environment array.

[code]
$env = $app-&gt;detectEnvironment(array(
	'local' =&gt; array('&lt;yourhostname&gt;','&lt;anotherhostname&gt;'),
));
[/code]

If you want to connect to a MySQL database you also of to update the local environment database config file `<project name>\app\config\local\database.php`.

**5. Run the Application**

Now we are ready to run the whole project. You can start the development server by typing `Grunt` into your command line. Navigate to [http://localhost:8000](http://localhost:8000) and you should see a simple Boostrap site.