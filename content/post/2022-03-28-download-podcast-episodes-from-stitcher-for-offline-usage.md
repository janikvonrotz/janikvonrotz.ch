---
title: "Download podcast episodes from Stitcher for offline usage"
slug: download-podcast-episodes-from-stitcher-for-offline-usage
date: 2022-03-28T13:58:54+02:00
categories:
 - Knowledge
tags:
 - podcast
 - offline
 - download
images:
 - /images/data-tiles.png
---

I love the podcast [Philosophize This!](https://www.philosophizethis.org/). It is freely available, but there is no meaningful way to download all episodes without using a third party app. As a data hoarder I want to have every episode on my hard disk. Therefore I created a simple script to download the [episodes from Stitcher](https://www.stitcher.com/show/philosophize-this).

<!--more-->

Here is the script:

```bash
# Get list of all episodes
curl 'https://api.prod.stitcher.com/show/philosophize-this/latestEpisodes?count=200' | jq '.' > podcast.json
# Process data and download episode if not downloaded yet
jq -c '.data.episodes[] | {title: .title, slug: .slug, url: .audio_url}' podcast.json | while read e; do 
    TITLE=$(echo "$e" | jq -r .title)
    SLUG=$(echo "$e" | jq -r .slug)
    URL=$(echo "$e" | jq -r .url)

    if [ -f "$SLUG.mp3" ]; then
        echo "File $SLUG.mp3 already exists."
    else
        echo "Download $TITLE ..."
        curl -L "$URL" -o "$SLUG.mp3" --progress-bar
    fi
done
```

And here it is in action:

```bash
janikvonrotz@pop-os:~/Downloads$ ./podcast.sh                               
File episode-163-the-creation-of-meaning-escape-from-evil-90885817.mp3 already exists.
File episode-162-the-creation-of-meaning-the-denial-of-death-89963729.mp3 already exists.
File episode-161-karl-popper-the-open-society-and-its-enemies-89711371.mp3 already exists.
File episode-160-the-creation-of-meaning-kierkegaard-silence-obedience-and-joy-89488860.mp3 already exists.
File episode-159-the-creation-of-meaning-nietzsche-amor-fati-88209087.mp3 already exists.
File episode-158-the-creation-of-meaning-nietzsche-the-ascetic-ideal-87305371.mp3 already exists.
Download Episode #157 ... The Creation of Meaning - Beauvoir ...
######################################################################## 100.0%
Download Episode #156 ... Emil Cioran part 2 - Failure and Suicide ...
######################################################################## 100.0%
Download Episode #155 ... Emil Cioran - Absurdity and Nothingness ...
######################################################################## 100.0%
Download Episode #154 ... Pragmatism and Truth ...
########################################        
```

If the download fails at some point you can restart the script and it should pickup were it failed.
