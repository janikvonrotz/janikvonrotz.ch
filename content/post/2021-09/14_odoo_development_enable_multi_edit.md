---
title: "Odoo Development: Enable multi edit"
slug: odoo_development_enable_multi_edit
date: 2021-09-14T13:19:22+02:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

With Odoo 13 an new attribute was introduced to the tree view that enables editing multiple rows at once.

<!--more-->

Enable mutli edit is very simple. Simply add `multi_edit="1"` to the `<tree>` tag. Here is an example:

```xml
  ...
  <record model="ir.ui.view" id="document_class_list">
    <field name="name">Document Class List</field>
    <field name="model">certificate_planer.document_class</field>
    <field name="arch" type="xml">
      <tree limit="200" multi_edit="1">
        <field name="sequence" widget="handle"/>
        <field name="name"/>
        <field name="description"/>
        <field name="show_reason"/>
      </tree>
    </field>
  </record>
  ...
```

This tree viewwill provide multi edit for all fields. Check the records you want to update, set the value for one record and hit enter. A confirmation prompt will show up and asks wether you want to perform this action.

![](/images/odoo-development-enable-multi-edit.png)

Note: Allowing mass editing can have side effects. Multiple records can be updated at will and if not tracked such a change will go unnoticed.
