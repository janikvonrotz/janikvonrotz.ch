---
title: "Odoo Development: Group by related field"
slug: odoo_development_group_by_related_field
date: 2021-09-29T08:24:59+02:00
categories:
 - Odoo
tags:
 - odoo
 - development
 - tutorial
images:
 - /images/odoo_development.jpg
---

The Odoo ORM provides a [`read_group`](https://www.odoo.com/documentation/14.0/developer/reference/addons/orm.html?highlight=group#odoo.models.Model.read_group) method. Calling this method and passing a domain definition, fieldnames and a groupby fieldname will return a domain recordset grouped by the fieldname. A big drawback is that you cannot pass a relation to a subfield. Importing the `itertools` library gives us new options to do so.

<!--more-->

The examples are based on the Odoo App [Certificate Planner](https://github.com/Mint-System/Certificate-Planner).

In our scenario we  have these models and relations:

**change** `1:n`> **document revision** `n:1`> **document** `n:1`> **document type** `n:1`> **document class**

For each change we want to get all document revisions grouped by the document class.

The data aggregation happens inside a report. We assume that we already have a list of changes. In the code snippet below the document revisions are group by the related field `.document_id.type_id.class_id.name` using the `itertools.groupby` method and a lambda function.

```python
_logger = logging.getLogger(__name__)
import itertools
...

class MDLReport(models.AbstractModel):
	_name = 'report.certificate_planer.mdl_report'
	_description = 'Certificate Planner MDL Report'
		...
		
        # Get documents by change
        change_revisions = {}
        for change in changes:
            change_revisions[change.id] = {}
            # Group document revisions by document > document type > document class
            for key, items in itertools.groupby(change.revision_ids, lambda r: r.document_id.type_id.class_id.name):
                change_revisions[change.id][key] = list(items)
				
		_logger.info(change_revisions[change.id])
		
		...
```

The `groupby` method returns the a list of keys and its grouped items. Use `list()` to have ORM access on the list objects.

The `change_revisions` contains a key for each change id with a key of the group name as value. The group name key has the grouped items as value.

See the logger output for details:

```
odoo.addons.certificate_planer.report.mdl_report: {'Certification Document': [certificate_planer.document_revision(32676,), certificate_planer.document_revision(32882,), certificate_planer.document_revision(32515,), certificate_planer.document_revision(32684,), certificate_planer.document_revision(32685,)]}
odoo.addons.certificate_planer.report.mdl_report: {'Certification Document': [certificate_planer.document_revision(34451,), certificate_planer.document_revision(34453,), certificate_planer.document_revision(29489,), certificate_planer.document_revision(30137,), certificate_planer.document_revision(34454,)]}
odoo.addons.certificate_planer.report.mdl_report: {'Certification Document': [certificate_planer.document_revision(34592,), certificate_planer.document_revision(30049,), certificate_planer.document_revision(30051,), certificate_planer.document_revision(30055,)]}
odoo.addons.certificate_planer.report.mdl_report: {'Compliance Document': [certificate_planer.document_revision(32111,)], False: [certificate_planer.document_revision(31306,)]}
odoo.addons.certificate_planer.report.mdl_report: {'Certification Document': [certificate_planer.document_revision(30669,), certificate_planer.document_revision(30840,), certificate_planer.document_revision(30682,), certificate_planer.document_revision(31251,), certificate_planer.document_revision(31252,)]}
```

Iterate over data structure is easy in qweb:

```html
<t t-foreach="changes" t-as="change">
	<t t-foreach="change_revisions[change.id]" t-as="document_class">
		<span t-esc="document_class"/>
		<t t-foreach="document_class_value" t-as="revision">
			<span t-field="revision.document_id"/>
		</t>
	</t>
</t>
```

Note that using the `foreach` clause on a dictionary will set a `$as` variable with the key and a `$as_value` with its value.