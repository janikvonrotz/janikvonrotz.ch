---
title: "Hide native tabs with Tree Style tabs for Firefox"
slug: hide-native-tabs-with-tree-style-tabs-for-firefox
date: 2023-03-20T17:15:22+01:00
categories:
  - Web development
tags:
 - firefox
 - treestyle
 - tabs
 - css
images:
 - /images/firefox-treestyle-tab.png
---

As the workspace for developers has moved almost entirely into the browser, browser extensions have become an integral part for customizing the workflow. Recently, I started to the use the Tree Style addons: <https://addons.mozilla.org/en-US/firefox/addon/tree-style-tab/>

When using this extension, the native tab bar becomes obsolete. The following part of the post is an instruction on how to hide the native tab bar (which is a bit difficult).

<!--more-->

1. Type [about:support](about:support) into the address bar.
2. Look for the *Profile Folder* entry and click on *Open folder*.
3. Create a file named `chrome/userChrome.css` in the directory:

```bash
mkdir chrome && touch chrome/userChrome.css
```

4. Populate the file with the following CSS code:

```css
#main-window[tabsintitlebar="true"]:not([extradragspace="true"]) #TabsToolbar>.toolbar-items {
    opacity: 0;
    pointer-events: none;
}

#main-window:not([tabsintitlebar="true"]) #TabsToolbar {
    visibility: collapse !important;
}

#sidebar-box[sidebarcommand="treestyletab_piro_sakura_ne_jp-sidebar-action"] #sidebar-header {
    display: none;
}

.tab {
    margin-left: 1px;
    margin-right: 1px;
}
```

5. Then type [about:config] into the address bar and set `toolkit.legacyUserProfileCustomizations.stylesheets` to `true`.
6. Restart Firefox

Now the native tab should be hidden. To reset the change simply remove the `userCrhome.css` file.