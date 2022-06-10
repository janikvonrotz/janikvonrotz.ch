---
title: "Odoo Development: Show last updated date above list view"
slug: odoo-development-show-last-updated-date-above-list-view
date: 2022-06-08T10:01:06+02:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

Odoo runs various reports in the background. Some of these report take a while to finish and therefore seeing the last updated date is important. In a few steps I will show how you can add the last updated date to a list view.

![](/images/odoo-development-last-updated.png)

<!--more-->

First we need a placeholder for the date that is updated once the list view is loaded. Add the following list view button element to the Odoo module.

**static/src/xml/listview_last_updated_date.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<template xml:space="preserve">
  <t t-extend="ListView.buttons">
    <t t-jquery="div.o_list_buttons" t-operation="append">
      <div class="o_list_selection_box container_last_updated_on" t-if="widget.modelName == 'critical.forecast'">
        <i><strong>Last Updated On: <span class="container_last_updated_date"/></strong></i>
      </div>
    </t>
  </t>
</template>
```

Included the file in the `__manifest__.py`.

Next extend the JavaScript List Controller with the following file:

**static/src/js/show_last_updated_date.js**

```js
odoo.define('$MODULE_NAME.show_last_updated_date', function (require) {
    "use strict"

    var ListController = require('web.ListController')
    var session = require('web.session')
    var ulang = session.user_context['lang'].replace('_','-')
    
    ListController.include({
        renderButtons: function($node) {
            this._super(...arguments)
            if(this.$buttons.find('.container_last_updated_on').length) {
                this.set_last_update_date()
            }
         },
         updateButtons() {
            this._super(...arguments)
            if(this.$buttons.find('.container_last_updated_on').length) {
                this.set_last_update_date()
            }
        },
        set_last_update_date: function () {
            return this._rpc({
                model: this.modelName,
                method: 'search_read',
                args: [[], ['write_date']],
                kwargs: {
                    limit: 1,
                },
            }).then((result) => {
				if (result.length > 0) {
                    var write_date = new Date(result[0].write_date)
                    write_date = new Date(write_date.setMinutes(write_date.getMinutes() - write_date.getTimezoneOffset()))
                    if (result.length) {
                        this.$buttons.find('.container_last_updated_on').removeClass('d-none')
                        this.$buttons.find('.container_last_updated_date').text(write_date.toLocaleString(ulang))
                    } else {
                        this.$buttons.find('.container_last_updated_on').addClass('d-none')
                    }
                }
            })
        },
    })
})

```

This extension will retrieve the last write date of the current model, convert it to the users timezone and locale and then display it in the placeholder.

The JavaScript file must be included in the assets backend template.

```xml
<template id="button_render_js" inherit_id="web.assets_backend">
	<xpath expr="." position="inside">
		<script src="$MODULE_NAME/static/src/js/show_last_updated_date.js" type="text/javascript"/>
	</xpath>
</template>
```

Once these files are in place, the last write of the model records should be set automatically.