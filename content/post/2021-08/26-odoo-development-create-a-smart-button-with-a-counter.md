---
title: "Odoo Development: Create a smart button with a counter"
slug: odoo-development-create-a-smart-button-with-a-counter
date: 2021-08-26T08:29:56+02:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

Smart buttons help navigating between linked models in Odoo. Creating a smart button requires three components: counter field, compute function, view action and a button box on the view.

<!--more-->

The examples are based on the Odoo App [Certificate Planner](https://github.com/Mint-System/Certificate-Planner).

We are going to add a smart button to the form view of the `change` model. This requires a counter field on the model. We assume that the required relational fields already exists.

**models/change.py**

```py
from odoo import models, fields

class Change(models.Model):
    ...
	
    revision_count = fields.Integer(compute='_compute_revision_count')
	...
	
	revision_ids = fields.One2many("certificate_planer.document_revision", "change_id", string="Document Revisions")
	...
	
	def _compute_revision_count(self):
        for record in self:
            record.revision_count = len(self.revision_ids)
	...
	
	def view_revision_ids(self):
        self.ensure_one()
        return {
            "type": "ir.actions.act_window",
            "name": "Document Revisions",
            "view_mode": "tree,form",
            "res_model": "certificate_planer.document_revision",
            "domain": [("change_id", "=", self.id)],
            "context": "{'create': False}",
        }
```

So in order to have a smart button four things are required:

1. counter field
2. relation field to count
3. count compute function
4. view action for the smart button

The change on the form view simple.

**views/change.xml**

```xml
<odoo>
  ...

  <record model="ir.ui.view" id="change_form">
    <field name="name">Change Form</field>
    <field name="model">certificate_planer.change</field>
    <field name="arch" type="xml">
      <form string="Change Form">
        ...
        <sheet>
          <div class="oe_button_box" name="button_box">
            <button name="view_revision_ids" type="object" class="oe_stat_button" icon="fa-files-o">
              <field string="Document Revisions" name="revision_count" widget="statinfo"/>
            </button>
          </div>
          ...
        </sheet>
      </form>
    </field>
  </record>

</odoo>

```

Simply add the button box div after `<sheet>` tag. The result in this case is:

![odoo-development-smart-button](/images/odoo-development-smart-button.png)