---
title: "Odoo Development: Show message dialog"
slug: odoo-development-show-message-dialog
date: 2022-02-01T09:21:25+01:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

Some things are easily done in the Odoo framework, others are more difficult. An easy thing such as opening a dialog window with a custom message is more difficult than expected.

<!--more-->

This example is based on the Odoo App [Certificate Planner](https://github.com/Mint-System/Certificate-Planner).

In this scenario there is a method `store_tpi_report` that generates two pdf reports. Once the reports is generated a message as showed below is displayed: 

![](/images/odoo-message-dialog.png)

The first thing that is required is the dialog window definition.

**views/document.xml**

```xml
  ...

  <record id="document_message" model="ir.ui.view">
      <field name="name">Document Message</field>
      <field name="model">certificate_planer.document.message</field>
      <field name="arch" type="xml">
          <form>
              <field name="message" readonly="True"/>
              <footer>
                  <button name="action_close" string="Ok" type="object" default_focus="1" class="oe_highlight"/>
              </footer>
          </form>
      </field>
  </record>

</odoo>
```

There is a message field and a close button.

The method returns an action that shows the dialog.

**models/document.py**

```py
...

class Document(models.Model):

    ...
    
    def store_tpi_report(self):
        self.ensure_one()

        # Render report
        pdf_content, content_type = self.env.ref('certificate_planer.tpi_report').render_qweb_pdf(self.id)
        pdf_content, content_type = self.env.ref('certificate_planer.mdl_report').render_qweb_pdf(self.id)

        # Return message
        message_id = self.env['certificate_planer.document.message'].create({'message': 'The reports have been generated. See attachments of this documents.'})
        return {
            'name': 'Message',
            'type': 'ir.actions.act_window',
            'view_mode': 'form',
            'res_model': 'certificate_planer.document.message',
            'res_id': message_id.id,
            'target': 'new'
        }
...
```

The *Document Message* requires a class definition.

**models/document.py**

```py
...

class DocumentMessage(models.TransientModel):
    _name = 'certificate_planer.document.message'
    _description = "Show Message"

    message = fields.Text(required=True)

    def action_close(self):
        return {'type': 'ir.actions.act_window_close'}
```

The  `action_close` method returns another action. To reload the page us this definition:

```py
    def action_close(self):
        return {   
            'type': 'ir.actions.client',
            'tag': 'reload'
        }
```

Of course the *Document Message* form can be reused. Simply create the message object and return the action to call the form.