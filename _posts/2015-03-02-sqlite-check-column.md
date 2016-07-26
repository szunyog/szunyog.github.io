---
layout: post
title:  "SQLite check if a column exists c#"
date:   2015-03-02 20:31:49
categories: csharp
tags: [sqlite]
description: A c# function to check if a column exists in a SQLite database.
---

[`PRAGMA table_info(table-name)`](http://www.sqlite.org/pragma.html#pragma_table_info) returns one row for each column in the named table. Columns in the result set include the column name, data type, whether or not the column can be NULL, and the default value for the column.

The following simple c# code using this statement and check if a column exists on a specified table.

{% highlight csharp %}

private bool CheckIfColumnExists(string tableName, string columnName)
{
   using(var conn = new SQLiteConnection("Data Source=mydb.sqlite;"))
   {
     conn.Open();
     var cmd = conn.CreateCommand();
     cmd.CommandText = string.Format("PRAGMA table_info({0})", tableName);

     var reader = cmd.ExecuteReader();
     int nameIndex = reader.GetOrdinal("Name");
     while (reader.Read())
     {
         if (reader.GetString(nameIndex).Equals(columnName))
         {
             conn.close();
             return true;
         }
     }
     conn.Close();
    }
    return false;
}

{% endhighlight %}

