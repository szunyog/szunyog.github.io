---
layout: post
title:  "MS SQL list foreign keys"
date:   2014-10-21 20:31:49
categories: scripts
tags: [sql]
---

Sometime it is usefull, to list all the foreign keys of a database, the following script does it:

{% highlight sql %}
SELECT
	t.name AS TableWithForeignKey,
	c.name AS ForeignKeyColumn,
	r.name AS ReferencedTable,
	rc.name AS ReferencedColumnName
FROM sys.foreign_key_columns AS fk
	inner join sys.tables AS t ON fk.parent_object_id = t.object_id
	inner join sys.columns AS c ON (t.object_id = c.object_id  AND fk.parent_column_id = c.column_id)
	inner join sys.tables AS r ON fk.referenced_object_id = r.object_id
	inner join sys.columns AS rc ON (r.object_id = rc.object_id AND rc.column_id = fk.referenced_column_id)
ORDER BY
	TableWithForeignKey,
	fk.constraint_column_id
{% endhighlight %}

