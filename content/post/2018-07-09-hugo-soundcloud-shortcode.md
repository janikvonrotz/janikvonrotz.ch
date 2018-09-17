---
title: "Hugo Soundcloud shortcode"
date: 2018-07-09T16:52:17+02:00
author: Janik von Rotz
slug: hugo-soundcloud-shortcode
images:
  - /images/hugo static site generator logo.jpg
categories:
  - Hugo
  - Music
tags:
 - hugo
 - soundcloud
 - shortcode
 - embed
 - playlists
 - convert
 - markdown
---

The Hugo templating system is very flexible. Even when markdown falls short you can add simple snippets inside the content files. Hugo created [shortcodes](https://gohugo.io/content-management/shortcodes/) to circumvent the limitations. For my blog I have create a custom shortcode for embeding Soundcloud playlists into my markdown posts. Creating custom shortcodes is very easy. I will show you how it is done and moreover provide a tool to convert existing Soundcloud set urls into the custom Soundcloud shortcode.
<!--more-->

## Create a Souncloud shortcode 

Creating new shortcodes is as metioned very in easy.

This is an example for the Soundcloud playlist shortcode.

Create a new file **/layouts/shortcodes/soundcloud.html** with this content:

```html
<iframe 
    width="100%"
    height="450"
    scrolling="no"
    frameborder="no"
    allow="autoplay"
    src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/playlists/{{ index .Params 0 }}&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"
></iframe>
```

If this file exists, you can use it as a shortcode in your posts and pages:

```
{{</* soundcloud 8924360 */>}}
```

The parameter is a Soundcloud set id, which can be extracted from the embed options in the Soundcloud browser UI.

## Convert Soundcloud set links to Hugo shortcodes

If you have existing Soundcloud links and would like to convert them into shortcodes, follow these steps:

1. Login into your Soundcloud account.
2. Register a new Soundcloud app.
3. Store the client id of the app.
4. Replace the variable `_CLIENT_ID_` and `_SOUNDCLOUD_USERNAME_` in the script below.
5. Install the json command line tool [jq](https://stedolan.github.io/jq/).
6. Run the script in the posts folder.

```bash
selector="*.md"
username="_SOUNDCLOUD_USERNAME_"
clientId="_CLIENT_ID_"

# loop through all files that match the selector
for filename in $selector; do

    echo "Processing: $filename"

    soundcloudUrl=$(grep -oP "^https?://soundcloud.com/$username/sets/[^\s]+" $filename)

    if [[  $soundcloudUrl ]] ; then
    
        echo "Retrieve playlist id for: $soundcloudUrl"
        playlistId=$(curl -L "http://api.soundcloud.com/resolve?url=$soundcloudUrl&client_id=$clientId" | jq -r '.id')

        echo "Got playlist id: $playlistId"

        sed -i "s,${soundcloudUrl},{{</* soundcloud ${playlistId} */>}},g" $filename
    fi
done
```

Enjoy your Soundcloud 🎵