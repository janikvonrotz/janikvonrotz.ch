---
title: "Odoo Development: Tree view with custom styles"
slug: odoo-development-tree-view-with-custom-styles
date: 2022-09-26T09:13:47+02:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

The Odoo UI cannot be customized as easy as reports and websites. Using modules we can easily inject custom CSS styles.

<!--more-->

Here is an example where we set the width of an unlink button in a custom tree view.

**views/bom.xml**

```xml
<field name="part_ids" context="{'default_certificate_planer_bom_id': id}">
  <tree limit="200" class="button_width" delete="0">
	<field name="sequence" widget="handle"/>
	<field name="certificate_planer_part_id"/>
	<field name="designation"/>
	<button name="unlink" class="oe_edit_only oe_link" icon="fa-times" type="object" string="Remove"/>
  </tree>
</field>
```

The button has the name `unlink` and its parent tree elemnt has the CSS class `button_width`. The name and class are used as selectors in our custom CSS:

**static/src/css/style.css**

```css
.button_width [data-name="unlink"] {
    width: 32px;
}
```

Make sure that the `style.css` file is added to the module manifest using the `web.assets_backend` as reference.