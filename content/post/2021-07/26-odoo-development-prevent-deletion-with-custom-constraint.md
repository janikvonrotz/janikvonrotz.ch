---
title: "Odoo Development: Prevent deletion with custom constraint"
slug: odoo-development-prevent-deletion-with-custom-constraint
date: 2021-07-26T10:09:50+02:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

By default Odoo asks for confirmation if a record is being deleted. Making validations before deletion is up to the developer. I will show how you can add a custom constraint that prevents a record being deleted.

<!--more-->

The examples are based on the Odoo App [Certificate Planner](https://github.com/Mint-System/Certificate-Planner).

In our scenario a record from model `post_certification_item` can only be deleted if the field `change_id` is not set.

Extend the default `unlink` (aka delete) method.

**models/post_certification_item.py**

```py
from odoo import models, fields, api, _
from odoo.exceptions import UserError

class PostCertificationItem(models.Model):
    ...
	
    def unlink(self):
        if self.change_id:
            raise UserError(_('You cannot delete a Post Certification Item that links to a Change.'))
        return super(PostCertificationItem, self).unlink()

```

If you try to delete a record that links to a change and confirm the prompt, this message will pop up:

![](/images/odoo-development-custom-constraint.png)