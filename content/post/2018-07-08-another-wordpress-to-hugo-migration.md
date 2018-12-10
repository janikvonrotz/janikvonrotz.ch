---
title: Another Wordpress to Hugo migration
date: 2018-07-08T09:11:28+00:00
author: Janik von Rotz
slug: another-wordpress-to-hugo-migration
images:
  - /images/wordpress to hugo migration.png
categories:
  - Content management systems
tags:
 - wordpress
 - migration
 - hugo
 - cms
---
Using Wordpress to run this blog has become an overkill. It was time to change. Static site generators, no database content management systems and JavaScript based tools make Wordpress an deprecated blogging platform. I had the idea of moving away from Wordpress for a while and decided to take a chance two weeks ago. This post is about the migration process and the design decisions made in preparation of the migration. 

From the title you can tell that I chose Hugo as my next CMS, but why Hugo? First of all, I only have had a look at Jekyll and Hugo. Jekyll is written in ruby and Hugo in go. To make it short and simple, Hugo does whatever Jekyll does, but based on newer technologies and a lot faster. And also remember it doesn't really matter. Both Hugo and Jekyll use markdown files and have almost the same format to store posts and data. So no big difference except speed.
<!--more-->

To be shure it is worth the shot to move away from Wordpress, I came up with a few pro and counter arguments.

Why move a away from Wordpress?

* I do not want my markdown blog posts in a database.
* My blog was heavy integrated into the wordpress ecosystem. Too heavy in my opinion.
* I would like to have the advantages of a static site generator (git, files only, markdown, ...).
* Running Wordpress has heavy dependencies (php, mysql, nginx, ...).

What do I have to consider when ditching Wordpress?

* How can I migrate comments?
* How does the new platform manage media files?
* How can I convert the Wordpress shortcodes?
* What am I gonna do with the mail subscriber list, contact form and other Wordpress features?
* Is the migration a one-way-ticket to another platform?

In the end, these were only questions that to be answered and they have definitely already been answered by somebody else.

In the next sections I will pretend to do a Wordpress-to-Hugo-migration and guide you through to process.

# Export Wordpress Data

Every migration begins with an export of the source data. To export data from Wordpress we will use the [jekyll-exporter](https://wordpress.org/plugins/jekyll-exporter/) plugin. As the file format is very similar to Hugos, we can use this exporter to create the Hugo site.

Once the exporter plugin is installed on the Wordpress site, open a shell on the server and navigate to the jekyll-exporter plugin directory. Then run `php jekyll-export-cli.php > ~/export.zip`.

We run the export command from command line, because if run the export from the Wordpress admin page, you'll probably receive a timeout error .

Once the file has been exported, download and unzip it.

# Setup Hugo

Next we are going to set up the new Hugo site. For this of course you need to [install Hugo](https://gohugo.io/getting-started/installing/). IF done open your shell and enter these commands:

```bash
hugo new site _SITE_NAME_
git submodule add https://github.com/jrutheiser/hugo-lithium-theme themes/lithium
hugo serve
```

We now have running a Hugo site with the lithium theme. Easy wasn't it?

However, as there is nothing to show yet, Hugo will return a blank page.

Update the parameters in the `config.toml` file to finish the installation. Set the following attributes:

```
baseURL = "https://janikvonrotz.ch"
languageCode = "en-us"
title = "Janik von Rotz"
description = "Curious, Dedicated, Humble" 
theme = "hugo-lithium-theme"
```

I recommend to commit every change, when finishing a section from now on.

# Migration

In this section we are going through the steps it took to migrate the Wordpress content to Hugo.

Let's get started with the most important files - the post files.

## Posts

From the Jekyll export I copied all them markdown files from the `_posts` folder to `_SITE_NAME_/content/post`.

In order to preserve the Wordpress link structure `/YYYY/MM/DD/_SLUG_/`, you have to define a permalink structure in Hugo. This is a common use case:

**config.toml**

```toml
[permalinks]
  post = "/:year/:month/:day/:title/"
  page = "/:title/"
```

It is important to preserve the link structure, otherwise it will break referrers links pointing to your site.

## Pages

Migrating pages is very simple. Copy the page markdown files to `hugo-site/content/page` and you are done.

## Navigation

The navigation structure can be configured universally in Hugo.

Use the toml config below to define the navigation menu.

```toml
[menu]

  [[menu.main]]
    name = "Home"
    url = "/"
    weight = 1

  [[menu.main]]
    name = "Projects"
    url = "/projects/"
    weight = 2

  [[menu.main]]
    name = "Downloads"
    url = "/downloads/"
    weight = 3

  [[menu.main]]
    name = "Curriculum Vitae"
    url = "/cv/"
    weight = 4

  [[menu.main]]
    name = "Contact"
    url = "/contact/"
    weight = 5
```

Obviously I expect you to adapt my examples according to your use case.

## Encoded Characters

Some character might haven been encoded during the export process. They can be searched and replaced conveniently with the `sed` command from you shell:

```bash
selector="*.md"

# &lt; -> <
sed -i 's/&lt;/</g' $selector
# &gt; -> >
sed -i 's/&gt;/>/g' $selector
# &quot; -> "
sed -i 's/&quot;/"/g' $selector
# &amp; -> &
sed -i 's/\&amp;/\&/g' $selector
# &#8211; -> -
sed -i 's/&#8211;/-/g' $selector
# &#8217; -> ’
sed -i 's/&#8217;/’/g' $selector
# &#039; -> ’ (get rid of ')
sed -i "s/&#039;/’/g" $selector

```

## Contact Form

Wordpress shortcodes allow you to embed plugins or enriched content in posts and pages. Some shortcodes have to be removed as Hugo does not understand them.

Below is the shortcode for my contact form:

```
[contact-form to='contact@janikvonrotz.ch' subject='janikvonrotz.ch - contact form']
[contact-field label='Name' type='text' required='1'/]
[contact-field label='Email Adresse' type='email' required='1'/]
[contact-field label='Betreff' type='text'/]
[contact-field label='Nachricht' type='textarea'/]
[/contact-form]
```

As there is no Hugo server (only static files) that forward mail messages submitted by a contact form, I have to decided to get rid of the form entirely.

If you really need a contact form, I recommend to opt for [Formspree](https://formspree.io/).

## Code Fences

Wordpress uses shortcodes to separate code from the post content. With the script below you can search and replace Wordpress code blocks with markdown code fences.

```bash
selector="*.md"

# [code] -> ```
sed -i 's;\[code\];```;g' $selector
# [/code] -> ```
sed -i 's;\[\/code\];```;g' $selector
# <pre><code> -> ```
sed -i 's;<pre><code>;```;g' $selector
# </code></pre> -> ```
sed -i 's;</code></pre>;```;g' $selector

# [code lang="text"][code lang=text][code language="bash"][code lang="sql"]
# [code lang="css"][code lang=css][code lang="text"][code lang=text]
# [code lang="php"][code lang="html"][code lang="java"][code lang="xml"]
# [code lang="js"][code lang="powershell"]
# [code language="powershell"] -> ```_LANGUAGE_KEY_
sed -i -r 's;\[code (lang|language)="?([a-z]+)"?\];```\2;g'  $selector

# [code lang="shell"] -> ```bash
sed -i 's;\[code lang="shell"\];```bash;g' $selector
# [code language="shell"] -> ```bash
sed -i 's;\[code language="shell"\];```bash;g' $selector

# [code lang="javascript"] -> ```js
sed -i 's;\[code lang="javascript"\];```js;g' $selector
# [code language="javascript"] -> ```js
sed -i 's;\[code language="javascript"\];```js;g' $selector

# [code lang="ps"] -> ```powershell
sed -i 's;\[code lang="ps"\];```powershell;g' $selector
# [code language="ps"] -> ```powershell
sed -i 's;\[code language="ps"\];```ps;g' $selector
# [code lang="ps" highlight="1,3"] -> ```powershell
sed -i 's;\[code lang="ps"\ .*];```powershell;g' $selector

# <code> -> `
sed -i 's;<code>;`;g' $selector
# </code> -> `
sed -i 's;</code>;`;g' $selector

```

Syntax highlighting for code fences is disabled by default. Enable it by setting `pygmentsCodeFences = "true"` in your `config.toml` file.

## Shortcodes

If you want to embed content in markdown, you have to use an iframe. Having html tags in a markdown file is not very cool. That is way Hugo provides its own shortcodes. The bash commands below helps converting Wordpress shortcodes to Hugo shortcodes.

```bash
selector="*.md"

# [caption ...]<img src="..." alt="..." ... /> ...[/caption] -> {{</* figure src="..." title="..." */>}}
sed -i -r 's;\[caption .*\].*<img src="([^"]*)" .* />\s+(.*)\[/caption\];{{</* figure src="\1" title="\2" */>}};g' $selector

# [caption ...]<a href="..." ...><img src="..." alt="" ... /></a>...[/caption] -> {{</* figure src="..." title="..." */>}}
sed -i -r 's;\[caption .*\].*<img src="([^"]*)".*<\/a>\s+(.*)\[/caption\];{{</* figure src="\1" title="\2" */>}};g' $selector

# https://vimeo.com/64762621 -> {{</* vimeo _ID9_ */>}}
sed -i -r 's;^https?\:\/\/vimeo.com\/(.{9});{{</* vimeo \1 */>}};g' $selector

# https://vimeo.com/64762621 -> {{</* vimeo _ID8_ */>}}
sed -i -r 's;^https?\:\/\/vimeo.com\/(.{8});{{</* vimeo \1 */>}};g' $selector

# http://www.youtube.com/watch?v=_ID_ -> {{</* youtube _ID_ */>}}
sed -i -r 's;^(https?://)?(www\.youtube\.com)\/watch\?v=(.{11});{{</* youtube \3 */>}};g' $selector

# [video width="1280" height="716" mp4="..."][/video] -> <video width="1280" height="720" controls><source src="..." type="video/mp4">...</video>
sed -i -r 's;\[video width="(.+)" height="(.+)" (webm|mp4)="([^"]+)"\]\[\/video\];<video width="\1" height="\2" controls><source src="\4" type="video\/\3">Your browser does not support the video tag.</video>;g' $selector

# https://vimeo.com/64762621 -> {{< instagram  _ID8_ >}}
sed -i -r 's;^https?\:\/\/vimeo.com\/(.{8});{{< instagram  \1 >}};g' $selector
```

## Media

The Wordpress media manager is probably the biggest loss when switching to Hugo. You have to manage cropped images and other assets manually in Hugo.

Copy the `wp-content` folder to `hugo-site/static/`. Files in this folder will be available in the root path.

Optionally you can set a relative base path for the asset urls and convert `a` and `img` tags to its markdown syntax:

```bash
selector="*.md"

# https://janikvonrotz.ch/wp-content/uploads/ -> /wp-content/uploads/
sed -i 's;https:\/\/janikvonrotz.ch\/wp-content\/uploads\/;\/wp-content\/uploads\/;g' $selector

# <img src="..." alt="..." ... /> -> ![...](...)
sed -i -r 's;<img src="([^"]+)" alt="([^"]+)"[^\/]+\/>;!\[\2\]\(\1\);g' $selector
sed -i -r 's;<img src="([^"]+)" alt=""[^\/]+\/>;!\[Untitled]\(\1\);g' $selector
sed -i -r 's;<img class.+alt="([^"]+)" src="([^"]+)"[^\/]+\/>;!\[\1]\(\2\);g' $selector
sed -i -r 's;<img class.+src="([^"]+)".+alt="([^"]+)"[^\/]+\/>;!\[\2]\(\1\);g' $selector

# <a href="...">...</a> -> [...](...)
sed -i -r 's;^<a href="([^"]+)">(.+)<\/a>;\[\2\]\(\1\);g' $selector
sed -i -r 's;^<a href="([^"]+)" rel=".+">(.+)<\/a>;\[\2\]\(\1\);g' $selector

# http://janikvonrotz.ch to https://janikvonrotz.ch
sed -i -r 's;http:\/\/janikvonrotz.ch;https:\/\/janikvonrotz.ch;g' $selector
```

## Comments

We face the same problem with comments as we do with the mail contact form. There is no server logic available to process comments. Thus we have to use a JavaScipt based solution.

I recommend to use [JustComments](https://just-comments.com/). If this does not work for you, there is still [Disqus](https://disqus.com/).

Both services will provide JavaScript code which can be added to the Hugo footer template file.

I've decided to use Disqus. They make it quite it easy to [import the comments](https://help.disqus.com/import-export-and-syncing/importing-exporting) from Wordpress into their platform.

Hugo provides [internal templates](https://gohugo.io/templates/internal/) for the most common use cases for static websites. There is a Disqus template that can be enabled by setting the `disqusShortname = "_DISQUS_ID_"` parameter.

## Helpers

While exploring the ways of migrating my content, I've built and discarded various scripts and commands. In this section you'll find scripts that could be useful in dealing with other migration problems.

**Split-PostFilesByDate.sh**

Copies the posts into date separated folders.

```bash
selector="*.md"

# loop through all files that match the selector
for filename in $selector; do

    echo "Processing: $filename"

    # split file name into array by the minus delimiter
    IFS='-' read -r -a array <<< "$filename"

    # check if first element of array is a number
    re='^[0-9]+$'
    if [[ ${array[0]} =~ $re ]] ; then
       
        directory=${array[0]}/${array[1]}/${array[2]}

        # create a subfolder for the year, month and date
        echo "Create directory: $directory"
        mkdir -p "$directory"

        # create new filename
        newfilename=$(echo $filename | cut -c12-999)
        
        # move the file
        echo "Move file to new directory and name it: $newfilename"
        mv $filename  "$directory/$newfilename"
    fi
done

```

**Remove-Lines.sh**

Remove a matching line from all markdown files.

```bash
selector="*.md"

# Get rid of unnecessary meta fields such as guid, id or layout
sed -i '/guid: https:\/\/janikvonrotz.ch\/?p=/d' $selector
sed -i '/guid: https:\/\/janikvonrotz.ch\/?page_id=/d' $selector
sed -i '/id: [0-9]*/d' $selector
sed -i '/layout: \(post\|page\)/d' $selector

```

**Convert-PermalinkToSlug.sh**

Create Hugo slug from Jekyll permalink.

```bash
selector="*.md"

# permalink: /2017/08/06/js-snippet-set-tallest-height-on-siblings/ -> slug: js-snippet-set-tallest-height-on-siblings
sed -i -r 's;permalink: /[0-9]{4}/[0-9]{2}/[0-9]{2}/([^/]+)/;slug: \1;g' $selector
```

## Metadata Cleanup

The jekyll export posts and pages contain metadata that is not required for hugo. Below are the commands to get rid of the unsupported metadata.

```bash
selector="*.md"

# remove id property 
sed -i '/^id: .*/d' $selector

# remove layout property
sed -i '/^layout: .*/d' $selector

# remove guid property
sed -i '/^guid: .*/d' $selector

# remove disqus id property
sed -i '/^dsq_thread_id:/d' $selector
sed -i '/  - \"[0-9]\{10\}\"/d' $selector
```

## Not Covered

To get the site up and running I had to fix quite a lot of stuff. Here is a list of thing I did not cover in this guide:

* Rename some page titles equal to their file name
* Convert shortcodes that could not be regexed
* Delete posts
* Fix posts
* Copy Wordpress post drafts manually

In addition there are problems for which I have not found a suitable solution yet:

* IFTTT migration
* Broken link checker
* Soundcloud shortcode
* Search function
* Theme customization
* Jetpack mailing lists
* Featured post images

But I'm definitely going to tackle them in the near future.

# Conclusion

The preparation and execution of this Wordpress migration took me a while. Around 20 hours I assume. Not only I've learned about regex and its flavors, but also about about the common use cases for static websites. I'm glad the migration worked out well. There are still a lot things I have to work out, but I assume this is a good start.

Thanks your Wordpress for the good times 👍