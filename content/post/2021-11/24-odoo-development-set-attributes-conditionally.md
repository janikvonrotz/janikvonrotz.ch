---
title: "Odoo Development: Set attributes conditionally"
slug: odoo-development-set-attributes-conditionally
date: 2021-11-24T14:06:11+01:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

Attributes such as `invisible` can be set on fields, widgets and buttons based on a condition. By using the `attrs` attribute the field and condition can be passed to the element. Here is an example: `attrs="{'invisible': [('active', '=', True)]}"`.

But how do we achive this for `tree` elements where `attrs` is not available?

<!--more-->

The solution is simple, we are adding a inherited view that sets the property conditionally. In our scenario we want to enable the delete action only if a user is member of a specific group.

This example is based on the Odoo App [Certificate Planner](https://github.com/Mint-System/Certificate-Planner).

First set the `delete` property of tree element to `false`.

**views/specification.xml**

```xml
<odoo>
	
  <record model="ir.ui.view" id="specification_list">
    <field name="name">Specification List</field>
    <field name="model">certificate_planer.specification</field>
    <field name="arch" type="xml">
      <tree limit="200" delete="false">
        <field name="name"/>
      </tree>
    </field>
  </record>

  ...
```

Then add an inherited record.

```xml
  ...

  <record id="specification_delete_rule" model="ir.ui.view">
    <field name="name">specification_delete_rule</field>
    <field name="groups_id" eval="[(4,ref('group_certificate_planer_administrator'))]"/>
    <field name="model">certificate_planer.specification</field>
    <field name="inherit_id" ref="specification_list"/>
    <field name="arch" type="xml">
      <xpath expr="//tree" position='attributes'>
        <attribute name='delete'>true</attribute>
      </xpath>
    </field>
  </record>

</odoo>
```

With this strategy an condition can be applied to any view element.