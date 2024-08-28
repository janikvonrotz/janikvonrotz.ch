---
title: "Odoo Development:  Settings fields for your module"
slug: odoo-development-settings-fields-for-your-module
date: 2024-08-28T16:41:29+02:00
categories:
  - Software development
tags:
  - odoo
  - development
images:
  - /images/odoo_development.jpg
draft: false
---

Odoo modules store their settings on the settings page. The fields definitions for the settings are different from the module. These fields can be mapped with system parameters. I  would like to show how you can add settings field for your Odoo module. 

<!--more-->

In this case we have a module `meilisearch_base` that connects to the Meilisearch API. We store the API url and key in the settings page. 

Lets get started by adding new fields to the `res.config.settings` model.

**res_config_settings.py**

```python
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = "res.config.settings"

    module_meilisearch_base = fields.Boolean("Integrate with Meilisearch")
    meilisearch_api_url = fields.Char(
        "Meilisearch API Url", config_parameter="meilisearch.api_url"
    )
    meilisearch_api_key = fields.Char(
        "Meilisearch API Key", config_parameter="meilisearch.api_key"
    )
```

When the module `meilisearch_base` is installed the field `module_meilisearch_base`
 is true. Setting the field value to false will uninstall the module.

The settings field have `config_parameter` attribute. Setting a field value will automatically create a key/value entry in the `ir.config_parameter` table.

To display the the settings field we will extend the `base_setup.res_config_settings_view_form`.
 
 **res_config_settings_view.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">meilisearch_base.res_config_settings_view_form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="base_setup.res_config_settings_view_form" />
        <field name="arch" type="xml">
            <div name="integration" position="inside">

                <div
                    class="col-12 col-lg-6 o_setting_box"
                    id="meilisearch_base_setting"
                >
                    <div class="o_setting_left_pane">
                        <field name="module_meilisearch_base" />
                    </div>
                    <div class="o_setting_right_pane">
                        <label string="Meilisearch" for="module_meilisearch_base" />
                        <div class="text-muted">
                            Integrate with Meilisearch.
                        </div>
                        <div
                            attrs="{'invisible': [('module_meilisearch_base', '=', False)]}"
                        >
                            <div class="content-group mt16">
                                <label
                                    for="meilisearch_api_url"
                                    class="o_light_label"
                                />
                                <field name="meilisearch_api_url" />
                            </div>
                            <div class="content-group">
                                <label
                                    for="meilisearch_api_key"
                                    class="o_light_label"
                                />
                                <field name="meilisearch_api_key" />
                            </div>
                        </div>
                    </div>
                </div>

            </div>
        </field>
    </record>
</odoo>
```

The settings box is inserted in the `integration` section. To do so the module must depend on `base_setup`. This edit will produce this result:

![](/images/Odoo%20Settings%20Page%20Meilisearch.png)

The values can be loaded from a demo data file.

**meilisearch_index_demo.xml**

```xml
<odoo>

    <record id="demo_config_meilisearch_api_url" model="ir.config_parameter">
        <field name="key">meilisearch.api_url</field>
        <field name="value">http://meili01:7700</field>
    </record>

    <record id="demo_config_meilisearch_api_key" model="ir.config_parameter">
        <field name="key">meilisearch.api_key</field>
        <field name="value">test</field>
    </record>

    <record id="demo_index0" model="meilisearch.index">
        <field name="name">Countries</field>
        <field name="index_name">countries</field>
        <field name="model_id" ref="base.model_res_country" />
    </record>

</odoo>
```

The values are defined in the settings model but defined in the system parameter model. When opening the the system parameter you'll find the entries.

![](/images/Odoo%20System%20Parameter%20Meilisearch.png)

Now using the config values in your code is simple. You call the system parameter model with sudo and retrive value by its key: `self.env["ir.config_parameter"].sudo().get_param("meilisearch.api_url")`

Here is an example:

**meilisearch_index.py**

```python
import json
import logging

import meilisearch

from odoo import _, api, fields, models
from odoo.exceptions import UserError

_logger = logging.getLogger(__name__)


class MeilisearchIndex(models.Model):
    _name = "meilisearch.index"
    _description = "Meilisearch Index"

	...

    def get_meilisearch_client(self):
        icp = self.env["ir.config_parameter"].sudo()
        return meilisearch.Client(
            url=icp.get_param("meilisearch.api_url"),
            api_key=icp.get_param("meilisearch.api_key"),
            timeout=10,
        )
```

You can view the full code of this example here: <https://github.com/Mint-System/Odoo-Apps-Connector/>
