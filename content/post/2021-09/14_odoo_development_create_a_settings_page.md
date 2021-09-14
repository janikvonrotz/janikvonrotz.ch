---
title: "Odoo Development: Create a settings page"
slug: odoo_development_create_a_settings_page
date: 2021-09-14T16:36:23+02:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

Odoo stores all module settings in the `res.config.settings` model. This model is based on the `models.TransientModel` class, which means that the settings data is not persisted with this model. 

This is just nice to know. I will show how you can create a settings page for your Odoo module and let you know about its limitations.

<!--more-->

The examples are based on the Odoo App [Certificate Planner](https://github.com/Mint-System/Certificate-Planner).

We are going to complete four actions:
1. Inherit the settings model
2. Add settings view
3. Add settings menu and action
4. Get the settings param

## Inherit the settings model

Creating new settings fields is simple. Inherit the `res.config.settings` model and add new fields.

**models/res_config_settings.py**

```py
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    title_page_text = fields.Char(string='Title Page Text',
        config_parameter='certificate_planer.title_page_text',
        default="This document contains confidential information and is proprietary to Example.")
    
    footer_text = fields.Char(string='Footer Text',
        config_parameter='certificate_planer.footer_text',
        default="Copyright by Example")
```

Not the `config_parameter` attributes. This will ensure that the fields value is also stored in the `ir.config_parameter` model.

In our example we have two configurable text fields. The usage of field types are limited to boolean, integer, float, char, selection or many2one.

## Add settings view

Next we create the settings view.

**views/res_config_settings.xml**

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>

    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">certificate_planer.res.config.settings.form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <xpath expr="//div[hasclass('settings')]" position="inside">
                <div class="app_settings_block" data-string="Certificate Planner" string="Certificate Planner" data-key="certificate_planer">
                    
                    <h2>Demand Planner</h2>
                    <div class="row mt16 o_settings_container" id="certificate_planer_setting_container">
                        <div class="col-12 col-lg-12 o_setting_box" id="certificate_planer_text_block">
                            
                            <div class="o_setting_right_pane">
                                <div class="text-muted">
                                    Text blocks for MDL report.
                                </div>
                                <div class="content-group">

                                    <div class="row mt16">
                                        <label class="col-lg-3 o_light_label" string="Title Page Text" for="title_page_text"/>
                                        <field name="title_page_text" class="oe_inline" style="width: 70% !important;"/>
                                    </div>

                                    <div class="row mt16">
                                        <label class="col-lg-3 o_light_label" string="Footer Text" for="footer_text"/>
                                        <field name="footer_text" class="oe_inline" style="width: 70% !important;"/>
                                    </div>

                                </div>
                            </div>

                        </div>
                    </div>

                </div>
            </xpath>
        </field>
    </record>

</odoo>
```

Wow, that's very bloated. This view definition describes the settings menu on the left side and the parameters on the right side of the settings page. Orient yourself by having a look at the rendered page:

![](/images/odoo-development-settings-page.png)

## Add settings menu and action

We have the setting fields and the view, now we can add an action that opens the settings view and a menu entry to call the action.

**views/menu.xml**

```xml
<odoo>
  ...

  <record id="settings_action" model="ir.actions.act_window">
    <field name="name">Settings</field>
    <field name="type">ir.actions.act_window</field>
    <field name="res_model">res.config.settings</field>
    <field name="view_mode">form</field>
    <field name="target">inline</field>
    <field name="context">{'module': 'certificate_planer'}</field>
  </record>
  ...

  <menuitem name="Settings"
    id="settings_menu"
    sequence="0"
    parent="certificate_planer.configuration_menu"
    action="certificate_planer.settings_action"/>
  ...

</odoo>
```

## Get the settings param

Finally, I'll give you an example on how to retrieve a stored settings value.

**report/document_report.py**

```py
return {
	...
	'title_page_text': self.env['ir.config_parameter'].sudo().get_param('certificate_planer.title_page_text'),
	'footer_text': self.env['ir.config_parameter'].sudo().get_param('certificate_planer.footer_text')
}   
```

The settings values are stored in the `ir.config_parameter` model. Access permission checks must be skipped by using `sudo()`. The get param name is equal to  `config_parameter` attribute of the field definition.
