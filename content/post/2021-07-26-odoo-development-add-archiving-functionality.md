---
title: "Odoo Development: Add archiving functionality"
slug: odoo-development-add-archiving-functionality
date: 2021-07-26T09:58:30+02:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

Odoo has a builtin archiving function for any model. Once a record is archived it will not be displayed in any views and selections, but can is easily be unarchived. This functionality ensures that relations must not be removed if linked records is obsolete.

<!--more-->

The examples are based on the Odoo App [Certificate Planner](https://github.com/Mint-System/Certificate-Planner).

We are going to add the archiving function to our model `change_status`. First simply add the well-known field `active` to the model class.

**models/change_status.py**

```py
from odoo import models, fields, api, _

class ChangeStatus(models.Model):
    ...
	
    active = fields.Boolean(default=True)
```

This will enable the archive functionality.

If you have a custom search and form view, ensure that the archive filter is displayed.

**views/change_status.xml**

```xml
<odoo>

  <record model="ir.ui.view" id="change_status_search">
    <field name="name">Change Status Search</field>
    <field name="model">certificate_planer.change_status</field>
    <field name="arch" type="xml">
      <search>
        <field name="designation"/>
        <filter string="Archived" name="inactive" domain="[('active', '=', False)]"/>
      </search>
    </field>
  </record>
	
  ...
```

And show a ribbon if the record shown is archived.

```xml
  ...

  <record model="ir.ui.view" id="change_status_form">
    <field name="name">Change Status Form</field>
    <field name="model">certificate_planer.change_status</field>
    <field name="arch" type="xml">
      <form string="Change Status Form">
        <sheet>
          <widget name="web_ribbon" title="Archived" bg_color="bg-danger" attrs="{'invisible': [('active', '=', True)]}"/>
          <field name="active" invisible="1"/>
	      ...
```

The form and list actions to archive and unarchive the records will be generated automatically.

![](/images/odoo-development-change-status-archive.png)
