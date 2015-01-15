---
layout: post
title:  "MS SQL kill all other connections by database"
date:   2014-10-22 20:31:49
categories: scripts
tags: [sql]
---

The first trivial solution is to switch the database to single user mode. It can be done by the following command:

{% highlight sql %}
use master;
go
alter database [dbname]
set single_user
with rollback immediate;
go
{% endhighlight %}

And switch back to multi user mode:

{% highlight sql %}
alter database [dbname]
set multi_user;
GO
{% endhighlight %}


It is also possible to kill the database connections. In this case clients may connect or reconnect again after the script.

{% highlight sql%}
declare @db_name varchar(100) = 'dbname';
declare @kill_commands varchar(max) = '';

select
	@kill_commands=@kill_commands + 'kill ' + convert(varchar(5),spid) + ';'
from master..sysprocesses
where
	spid <> @@SPID -- avoid to kill the current process
	and dbid=db_id(@db_name)

print 'Executing: ' + @kill_commands;
exec (@kill_commands);
{% endhighlight %}

