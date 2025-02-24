---
title: "Dark mode support"
slug: dark-mode-support
date: 2019-10-11T16:23:26+02:00
categories:
 - Blog
tags:
 - dark mode
 - css
 - theme
images:
 - /images/dark-mode.png
---

Every website has to support dark mode now. Probably because of the new iPhone launch or so. Nonetheless, I added a bunch of CSS to this theme as well.
<!--more-->

```css
/* Dark Mode */

@media (prefers-color-scheme: dark) {
	body,
	body > header {
		background-color: #444;
		color: #e4e4e4;
	}

	p, li, nav a, label, input,
	h1, h2, h3, h4, h5, h6 {
		color: #e4e4e4;
	}
	a {
		color: #e39777;
	}
	img {
		filter: grayscale(30%);
	}
	nav .logo img,
	nav .search img {
		filter: invert(1)
	}

	/* Article */

	article blockquote {
		background-color: #444;
	}

	/* Archive */

	section.archive a {
		color: #e39777;
	}
	section.archive .link:hover {
		background: #cbf8fc;
		border-bottom-color: #cbf8fc;
	}
}
```

Enable dark mode on your device and fancy the view.