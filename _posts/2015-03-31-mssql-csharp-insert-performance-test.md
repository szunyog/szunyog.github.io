---
layout: post
title:  "MS SQL - C# insert performance test"
date:   2015-03-31 20:31:49
categories: csharp
tags: [mssql, csharp]
---

## Introduction

The structure of the database (foreign keys, indexes, etc...) has the biggest effect to the database performance. But it is also does matter how the client side is implemented. The .Net framework supports several ways to implement database insert operations. In certain scenarios it is also possible to reach huge performance increase by choosing the appropriate method.

In this tests I am using a time based curve data to insert. The tests are focused to determine which way is the best to insert several thousands records at one time.

During the test I am using .Net framework 4.5 and Microsoft SQL server 2008  Express edition.

## Data structure

The .Net side data structure is the following:

{% highlight csharp %}

public class CurveData
{
    public int CurveId { get; set; }
    public DateTime TimeStamp { get; set; }
    public Decimal Value { get; set; }
}

{% endhighlight %}

And the SQL side looks like this:

{% highlight sql %}

CREATE TABLE [dbo].[CurveData](
	[CurveId] [int] NOT NULL,
	[TimeStamp] [datetime] NOT NULL,
	[Value] [decimal](23, 6) NOT NULL,
 CONSTRAINT [PK_dbo.CurveDatas] PRIMARY KEY CLUSTERED
(
	[CurveId] ASC,
	[TimeStamp] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]

{% endhighlight %}

The primary key and the clustered index is the CurveId and TimeStamp field.

## Tests

### Test 1: Constructed SQL statement without paramters

In this case the whole sql statement is constructed in c# code, including the values so the SQL server will not have chance to use cached version of the complied sql statement.

{% highlight csharp %}

public void InsertOneConcatenatedSqlTest(List<CurveData> data)
{
    System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
    watch.Start();

    using (SqlConnection connection = new SqlConnection(ConnectionString))
    {
        connection.Open();
        using (SqlTransaction tran = connection.BeginTransaction())
        {
            var insertCommand = connection.CreateCommand();
            insertCommand.Transaction = tran;
            insertCommand.CommandType = System.Data.CommandType.Text;
            foreach (var item in data)
            {
                insertCommand.CommandText = string.Format("INSERT INTO dbo.CurveData ([CurveId], [TimeStamp], [Value]) VALUES({0},'{1}', {2})", item.CurveId, item.TimeStamp, item.Value);
                insertCommand.ExecuteNonQuery();
            }
            tran.Commit();
        }
    }
    watch.Stop();
    Console.WriteLine("InsertOneConcatenatedSqlTest: {0} items saved in {1} ms.", data.Count, watch.ElapsedMilliseconds);
}
{% endhighlight %}

### Test 2: Constructed SQL statement with paramters

In this case the whole sql statement is constructed in c# code, but the values are passed as paramters so the SQL server will have a chance to use cached version of the complied sql statement.

{% highlight csharp %}

public void ConstructedSQLWithParamters(List<CurveData> data)
{
    System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
    watch.Start();

    using (SqlConnection connection = new SqlConnection(ConnectionString))
    {
        connection.Open();
        using (SqlTransaction tran = connection.BeginTransaction())
        {
            var insertCommand = connection.CreateCommand();
            insertCommand.Transaction = tran;
            insertCommand.CommandText = "INSERT INTO dbo.CurveData ([CurveId], [TimeStamp], [Value]) VALUES(@CurveId, @TimeStamp, @Value)";
            insertCommand.CommandType = System.Data.CommandType.Text;

            var curveIdParamter = new SqlParameter("CurveId", SqlDbType.Int);
            var timeStampParamter = new SqlParameter("TimeStamp", SqlDbType.DateTime);
            var valueParamter = new SqlParameter("Value", SqlDbType.Decimal);
            insertCommand.Parameters.Add(curveIdParamter);
            insertCommand.Parameters.Add(timeStampParamter);
            insertCommand.Parameters.Add(valueParamter);

            foreach (var item in data)
            {
                curveIdParamter.Value = item.CurveId;
                timeStampParamter.Value = item.TimeStamp;
                valueParamter.Value = item.Value;
                insertCommand.ExecuteNonQuery();
            }
            tran.Commit();
        }
    }
    watch.Stop();
    Console.WriteLine("ConstructedSQLWithParamters: {0} items saved in {1} ms.", data.Count, watch.ElapsedMilliseconds);
}
{% endhighlight %}

### Test 3: Constructed SQL statement with multiple values

In this case the whole sql statement is constructed in c# code, but multiple values are used in one statement. In this case the SQL server cannot use cached queries, but the number of the sql operations could be decreased. In this test 100 items per sql statement is used.

{% highlight csharp %}

public void ConstructedSQLWithSeveralValues(List<CurveData> data)
{
    System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
    watch.Start();

    using (SqlConnection connection = new SqlConnection(ConnectionString))
    {
        connection.Open();
        using (SqlTransaction tran = connection.BeginTransaction())
        {
            var insertCommand = connection.CreateCommand();
            insertCommand.Transaction = tran;
            insertCommand.CommandType = System.Data.CommandType.Text;
            StringBuilder insertStatement = new StringBuilder();

            // max 1000 items/statement
            int counter = 0;
            foreach (var item in data)
            {
                if (counter == 0)
                {
                    insertStatement.Append("INSERT INTO dbo.CurveData ([CurveId], [TimeStamp], [Value]) Values ");
                }
                insertStatement.Append(string.Format("({0}, '{1}', {2}),", item.CurveId, item.TimeStamp, item.Value));

                counter++;
                if (counter == 100)
                {
                    //remove last coma
                    insertStatement.Length--;
                    insertCommand.CommandText = insertStatement.ToString();
                    insertCommand.ExecuteNonQuery();
                    insertStatement.Clear();
                    counter = 0;
                }
            }

            if (counter > 0)
            {
                //remove last coma
                insertStatement.Length--;
                insertCommand.CommandText = insertStatement.ToString();
                insertCommand.ExecuteNonQuery();
                insertStatement.Clear();
                counter = 0;
            }
            tran.Commit();
        }
    }
    watch.Stop();
    Console.WriteLine("ConstructedSQLWithSeveralValues: {0} items saved in {1} ms.", data.Count, watch.ElapsedMilliseconds);
}
{% endhighlight %}

### Test 4: Stored procedure

In this case a stored procedure is used to insert the values. One procedure call is one insert operation.

{% highlight csharp %}

public void StoredProcedure(List<CurveData> data)
{

    System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
    watch.Start();

    using (SqlConnection connection = new SqlConnection(ConnectionString))
    {
        connection.Open();
        using (SqlTransaction tran = connection.BeginTransaction())
        {
            var insertCommand = connection.CreateCommand();
            insertCommand.Transaction = tran;
            insertCommand.CommandText = "dbo.InsertCurveData";
            insertCommand.CommandType = System.Data.CommandType.StoredProcedure;

            var curveIdParamter = new SqlParameter("CurveId", SqlDbType.Int);
            var timeStampParamter = new SqlParameter("TimeStamp", SqlDbType.DateTime);
            var valueParamter = new SqlParameter("Value", SqlDbType.Decimal);
            insertCommand.Parameters.Add(curveIdParamter);
            insertCommand.Parameters.Add(timeStampParamter);
            insertCommand.Parameters.Add(valueParamter);

            foreach (var item in data)
            {
                curveIdParamter.Value = item.CurveId;
                timeStampParamter.Value = item.TimeStamp;
                valueParamter.Value = item.Value;
                insertCommand.ExecuteNonQuery();
            }
            tran.Commit();
        }
    }
    watch.Stop();
    Console.WriteLine("StoredProcedure: {0} items saved in {1} ms.", data.Count, watch.ElapsedMilliseconds);
}
{% endhighlight %}

The source of the stoed procedure is quite simple:

{% highlight SQL %}

CREATE PROCEDURE dbo.InsertCurveData
    @CurveId int,
	@TimeStamp datetime,
	@Value decimal(23, 6)
AS
BEGIN
  SET NOCOUNT ON;

  INSERT INTO dbo.CurveData ([CurveId], [TimeStamp], [Value])
  VALUES(@CurveId, @TimeStamp, @Value)
END

GO

{% endhighlight %}




### Test 5: Stored procedure with structured paramter

From the version of 2008 the Microsoft SQL server support structured parameter. With this method it is possible to pass a structured list as a paramter to a stored procedure. It has a limitation that the parameter type must be  `DataTable`, `DbDataReader` or `System.Collections.Generic.IEnumerable<SqlDataRecord>`

In this case you need to define a custom data type in the SQL side:

{% highlight SQL %}

CREATE TYPE dbo.CurveDataType
AS TABLE
(
    [CurveId] int,
	[TimeStamp] datetime,
	[Value] decimal(23, 6)
);

GO
{% endhighlight %}

And a stored procedure with this type of paramter:

{% highlight SQL %}

CREATE PROCEDURE dbo.InsertCurveDataList
  @List AS dbo.CurveDataType READONLY
AS
BEGIN
  SET NOCOUNT ON;

  INSERT INTO dbo.CurveData ([CurveId], [TimeStamp], [Value])
  SELECT [CurveId], [TimeStamp], [Value] FROM @List;
END

go

{% endhighlight %}

And the C# code to invoke this stored procedure. Firstly the data should be converted to one of the above mentioned format, in this sample I tested with DataTable and DataRecord:

{% highlight csharp %}

public void StoredProcedureList(List<CurveData> data)
{
    System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
    watch.Start();

    using (SqlConnection connection = new SqlConnection(ConnectionString))
    {
        connection.Open();
        using (SqlTransaction tran = connection.BeginTransaction())
        {

            var insertCommand = connection.CreateCommand();
            insertCommand.Transaction = tran;
            insertCommand.CommandText = "dbo.InsertCurveDataList";
            insertCommand.CommandType = System.Data.CommandType.StoredProcedure;

            DataTable parameterTable = new DataTable();
            parameterTable.Columns.Add("CurveId", typeof(int));
            parameterTable.Columns.Add("[TimeStamp]", typeof(DateTime));
            parameterTable.Columns.Add("[Value]", typeof(decimal));

            foreach (var item in data)
            {
                parameterTable.Rows.Add(item.CurveId, item.TimeStamp, item.Value);
            }

            Console.WriteLine("StoredProcedureList: {0} items data converted in {1} ms.", data.Count, watch.ElapsedMilliseconds);

            SqlParameter tvparam = insertCommand.Parameters.AddWithValue("@List", parameterTable);
            tvparam.SqlDbType = System.Data.SqlDbType.Structured;
            insertCommand.ExecuteNonQuery();
            tran.Commit();
        }
    }
    watch.Stop();
    Console.WriteLine("StoredProcedureList: {0} items saved in {1} ms.", data.Count, watch.ElapsedMilliseconds);
}

public void InsertListTestSqlDataRecord(List<CurveData> data)
{
    System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
    watch.Start();

    using (SqlConnection connection = new SqlConnection(ConnectionString))
    {
        connection.Open();
        SqlMetaData[] meta = new SqlMetaData[] {
            new SqlMetaData("CurveId", SqlDbType.Int),
            new SqlMetaData("TimeStamp", SqlDbType.DateTime),
            new SqlMetaData("Value", SqlDbType.Decimal),
        };
        using (SqlTransaction tran = connection.BeginTransaction())
        {
            var insertCommand = connection.CreateCommand();
            insertCommand.Transaction = tran;
            insertCommand.CommandText = "dbo.InsertCurveDataList";
            insertCommand.CommandType = System.Data.CommandType.StoredProcedure;
            List<SqlDataRecord> records = new List<SqlDataRecord>();

            foreach (var item in data)
            {
                SqlDataRecord r = new SqlDataRecord(meta);
                r.SetValues(item.CurveId, item.TimeStamp, item.Value);
                records.Add(r);
            }

            Console.WriteLine("InsertListTestSqlDataRecord: {0} items data converted in {1} ms.", data.Count, watch.ElapsedMilliseconds);

            SqlParameter tvparam = insertCommand.Parameters.AddWithValue("@List", records);
            tvparam.SqlDbType = System.Data.SqlDbType.Structured;
            insertCommand.ExecuteNonQuery();
            tran.Commit();
        }
    }
    watch.Stop();
    Console.WriteLine("InsertListTestSqlDataRecord: {0} items saved in {1} ms.", data.Count, watch.ElapsedMilliseconds);

}

{% endhighlight %}

The last test case is to pass an XML document to the stored procedure and parse it. To generate the XML content the `WriteXml` methdod of the `DataTable` is used. *(There are faster ways to generate XML files, the test cases focuses on sql execution duration and not the parameter set up.)*


{% highlight csharp %}

public void StoredProcedureXml(List<CurveData> data)
{
    System.Diagnostics.Stopwatch watch = new System.Diagnostics.Stopwatch();
    watch.Start();

    using (SqlConnection connection = new SqlConnection(ConnectionString))
    {
        connection.Open();
        using (SqlTransaction tran = connection.BeginTransaction())
        {

            var insertCommand = connection.CreateCommand();
            insertCommand.Transaction = tran;
            insertCommand.CommandText = "dbo.InsertCurveDataXml";
            insertCommand.CommandType = System.Data.CommandType.StoredProcedure;

            DataTable parameterTable = new DataTable("data");
            parameterTable.Columns.Add("CurveId", typeof(int));
            parameterTable.Columns.Add("TimeStamp", typeof(DateTime));
            parameterTable.Columns.Add("Value", typeof(decimal));
            parameterTable.Columns[1].DateTimeMode = DataSetDateTime.Utc;
            foreach (var item in data)
            {
                parameterTable.Rows.Add(item.CurveId, item.TimeStamp, item.Value);
            }
            var test = parameterTable.Select("TimeStamp = 'Mar 27 2016  1:15AM'").ToList();

            using (StringWriter sw = new StringWriter())
            {
                parameterTable.WriteXml(sw);
                sw.Flush();
                SqlParameter tvparam = insertCommand.Parameters.AddWithValue("@data", sw.ToString());
                tvparam.SqlDbType = System.Data.SqlDbType.Xml;
            }

            Console.WriteLine("StoredProcedureXml: {0} items data converted in {1} ms.", data.Count, watch.ElapsedMilliseconds);

            insertCommand.ExecuteNonQuery();
            tran.Commit();
        }
    }
    watch.Stop();
    Console.WriteLine("InsertListTestXml: {0} items saved in {1} ms.", data.Count, watch.ElapsedMilliseconds);
}

{% endhighlight %}

And the stored procedure is like this:

{% highlight SQL %}
CREATE PROCEDURE dbo.InsertCurveDataXml
    @data xml
AS
BEGIN
  SET NOCOUNT ON;

  INSERT INTO dbo.CurveData ([CurveId], [TimeStamp], [Value])
  SELECT DISTINCT
    DataToInsert.data.value('(CurveId/text())[1]','int') as [CurveId],
    DataToInsert.data.value('(TimeStamp/text())[1]','datetime') as [TimeStamp],
	DataToInsert.data.value('(Value/text())[1]','decimal') as [Value]
    FROM @data.nodes('/DocumentElement/data') DataToInsert(data)
END

{% endhighlight %}

## Results

{% highlight sql %}
ConstructedSQLWithoutParamters: 35136 items saved in 4225 ms.
ConstructedSQLWithParamters: 35136 items saved in 2595 ms.
StoredProcedure: 35136 items saved in 2513 ms.
ConstructedSQLWithSeveralValues: 35136 items saved in 4044 ms.
StoredProcedureList: 35136 items data converted in 66 ms.
StoredProcedureList: 35136 items saved in 310 ms.
InsertListTestSqlDataRecord: 35136 items data converted in 123 ms.
InsertListTestSqlDataRecord: 35136 items saved in 499 ms.

{% endhighlight %}

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.8.2/jquery.min.js"></script>
<script src="http://code.highcharts.com/highcharts.js"></script>
<div id="container" style="width:100%; height:400px;"></div>

<script>
$(function () {
    $('#container').highcharts({
        chart: {
            type: 'bar'
        },
        title: {
            text: 'Results'
        },
        xAxis: {
		categories: ['ConstructedSQLWithoutParamters',
		'ConstructedSQLWithParamters',
		'StoredProcedure',
		'ConstructedSQLWithSeveralValues',
		'StoredProcedureList',
		'InsertListTestSqlDataRecord']
        },
        yAxis: {
            title: {
                text: 'ms'
            }
        },
        series: [{
				name: 'Time',
            data: [4225, 2595, 2513, 4044, 310, 499]
        }]
    });
});
</script>


