---
title: "Convert markdown wiki Links to html links"
slug: convert-markdown-wiki-links-to-html-links
date: 2020-09-03T11:35:27+02:00
categories:
 - Web Development
tags:
 - markdown
 - wiki
 - convert
 - javascript
 - regex
images:
 - /images/blue orange.jpg
---

[Wiki-Links](https://de.wikipedia.org/wiki/Hilfe:Links) `[[ ]]` are not part of the markdown specification, but are often used by markdown editors such as [Obsidian](https://obsidian.md/). As they are not supported by most markdown converters we need to convert the wiki links on our own.
<!--more-->

To generate beautiful documentations from my markdown files, I'm using [docsify](docsify.js.org/). Docsify is a singe page application that reads and converts the content from markdown files inside a folder. It allows to plugin code in the the markdown render process via hooks. The code I've written to convert wiki links can be used for other platforms as well. So there is no need to use docsify in order to understand what is going on.

Let's have a look on how I have converted to wiki links to html a-tags. Create a folder on the command line and have the latest `node` version at your hand.

Create JavaScript file as showed below:

**convert.js**

```js
const convert = function (content) {

    // convert wiki image links
    const wikiImage = /!\[\[([^\]]*)\]\]/g
    content = content.replace(wikiImage, '<img src="/assets/$1" \/>')

    // convert wiki links
    const link = /\[\[([^\]]*)\]\]/g
    let matches = content.match(link) || []
    for (i = 0; i < matches.length; i++) {

        let match = matches[i]
        let title = match.match(/\[\[([^\]]*)/)[1]
        let href, anchor

        if (match.indexOf('#') > 0) {
            anchor = match.match(/#([^\||\]]*)/)[1]
            // Convert to docsify id
            anchor = anchor.toLowerCase().replace(/ /g, "-")
        }

        if (match.indexOf('|') > 0) {
            href = match.match(/\[\[([^\||#]*)/)[1]
            title = match.match(/\|(.*)\]\]/)[1]
        }

        // build the a-tag
        content = content.replace(match, `<a href="/#/${href ? href : title}${anchor ? ('?id=' + anchor) : ''}">${title}</a>`)
    }

    return content
}
module.exports = convert
```

The `convert.js` export a function that expects markdown content and then converts all the wiki links.

Here follows short demonstration on how to use this function.

Create a markdown file in the same folder.

**content.md**

```md
[[Lorem ipsum#dolor|Lorem dope]] dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est [[Lorem ipsum]] dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et [[dolore|lorem]] magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est [[#dolor|Lorem]] ipsum dolor sit amet.
```

And this test file as well.

**test.js**

```js
fs = require('fs')
convert = require('./convert')

fs.readFile('./content.md', 'utf8', function (err, data) {
    if (err) {
        return console.log(err);
    }
    console.log(convert(data))
})

```

Finally, run the test with the command `node test.js` and you should get these results:

```txt
<a href="/#/Lorem ipsum?id=dolor">Lorem dope</a> dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est <a href="/#/Lorem ipsum">Lorem ipsum</a> dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et <a href="/#/dolore">lorem</a> magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est <a href="/#/Lorem?id=dolor">Lorem</a> ipsum dolor sit amet.
```

The links have been converted according to my custom a-tag build.
