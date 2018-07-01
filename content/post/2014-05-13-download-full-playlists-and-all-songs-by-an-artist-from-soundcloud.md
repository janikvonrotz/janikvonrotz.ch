---
id: 2235
title: Download full playlists and all songs by an artist from SoundCloud
date: 2014-05-13T07:53:13+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2235
permalink: /2014/05/13/download-full-playlists-and-all-songs-by-an-artist-from-soundcloud/
dsq_thread_id:
  - "2681691439"
image: /wp-content/uploads/2013/07/soundcloud.png
categories:
  - Music
  - Ubuntu Server
tags:
  - album
  - all
  - artist
  - download
  - full
  - git
  - mac
  - offline
  - osx
  - playlist
  - script
  - songs
  - souncloud
  - type
  - ubuntu
---
# Introduction

First of all I want to apologise for what I'm writing about in this post.
SoundCloud is a music platform where you can listen to almost every song for free.
This is a great network and only works as long users don't download the songs from SoundCloud.
Great artists advertise their content to get more attention, the last thing they want is that you'll download their songs.
However there's no way to prevent people from doing so, as you will see not even the guys behind SoundCloud can prevent their content from being downloaded.
I'd ask you to donate for or buy the songs from the artists you like on SoundCloud.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [cURL, Git](https://janikvonrotz.ch/2014/03/25/install-ubuntu-packages/)

# Instructions

Clone the Soundcloud music downloader repository from GitHub.

	cd /usr/local/src
	sudo git clone https://github.com/lukapusic/soundcloud-dl.git

Run the the installer script. This will install the required ubuntu packages and create a config file in your home folder.

	cd soundcloud-dl/
	sudo ./install
	Set path : /home/[username]

Switch to your home folder and start downloading from a SoundCloud url.

	cd ~
	./usr/local/src/scdl [url]

