---
title: "Odoo Development: Add activity button"
slug: odoo_development_add_activity_button
date: 2021-07-14T10:07:47+02:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

The activity button is part of the chatter. Every model form can have activities. I will show how you can enable the activity button for your model form using the mixin class.

<!--more-->

The examples are based on the Odoo App [Certificate Planner](https://github.com/Mint-System/Certificate-Planner).

First inherit the classes in your model.

**models/change.py**

```py
class Change(models.Model):
	_inherit = ['mail.thread', 'mail.activity.mixin']
```

Next update the form.

**views/change.xml**

```xml
        </sheet>
        <div class="oe_chatter">
          <field name="message_follower_ids" widget="mail_followers"/>
          <field name="message_ids" widget="mail_thread"/>
          <field name="activity_ids" widget="mail_activity"/>
        </div>
      </form>
    </field>
  </record>

</odoo>
```

And you will see the message and activity buttons on your form.

![](/images/odoo_development_activity.png)