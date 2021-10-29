---
title: "Obsidian and Vuepress: include markdown files"
slug: obsidian-and-vuepress-include-markdown-files
date: 2021-10-29T19:21:40+02:00
categories:
 - Knowledge
tags:
 - obsidian
 - git
 - synchronization
images:
 - /images/obsidian-banner.png
---

I am using [Obisidan](https://obsidian.md/) to manage markdown files and [Vuepress](https://vuepress.vuejs.org/) to publish them. With the help of a script `build.js` the Obsidian vault is converted into a publishable format for Vuepress. Obsidian supports embeding markdown files using the `![](file.md)` syntax. Here I'll show you how this feature can be enabled for Vuepress.

<!--more-->

First we need to support including markdown files for Vuepress. Install the [markdown-it](https://github.com/markdown-it/markdown-it) extension.

```bash
npm install --save markdown-it-include
```

And enable it (Vuepress v1).

**.vuepress/config.js**

```js
module.exports = {
	// ...
    extendMarkdown: (md) => {
        md.use(require('markdown-it-include'))
    }
}
```

Before the Vuepress build is created the `build.js` file is executed.

```js
// ...

function convert(content,file) {
    // ...
	
    // convert include markdown links
    // ![title](file.md) -> !!!include(file.md)!!!
    const mdInclude = /(!\[.*?\]\(.*?\.md\))/g
    matches = content.match(mdInclude) || []
    for (i = 0; i < matches.length; i++) {
        let match = matches[i]
        
        let include = match.match(/!\[.*\]\((.*\.md)\)/)[1]
        include = sanitizeName(include)
                
        content = content.replace(match, `!!!include(${include})!!!`)
    }
	
	//...
	
	return content
}

// ...
```

This is just an excerpt. The script does a lot more. It renames files, moves assets, builds indexes and overall tweaks the markdown files to work with Vuepress.

Have look [here](https://github.com/Mint-System/Wiki/blob/master/build.js) for a full example.