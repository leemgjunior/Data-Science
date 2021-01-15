# SQL Server

#### Reference Material:

* [Use Python in SQL Server 2017 for advanced data analytics](https://www.sqlshack.com/how-to-use-python-in-sql-server-2017-to-obtain-advanced-data-analytics/)
  * [Handle inputs and outputs using Python in SQL Server 2017](https://docs.microsoft.com/en-us/sql/advanced-analytics/tutorials/quickstart-python-inputs-and-outputs?view=sql-server-2017)  
* [Running Docker MSSQL Server a Mac](https://medium.com/@reverentgeek/sql-server-running-on-a-mac-3efafda48861)
* [How to Handle Calculations Related to fiscal year and quarter](https://www.sqlservercentral.com/articles/how-to-handle-calculations-related-to-fiscal-year-and-quarter)
*  [How to Identify a Scheduled SSRS report in SQL Agent Jobs](https://www.mssqltips.com/sqlservertip/1846/how-to-easily-identify-a-scheduled-sql-server-reporting-services-report/)
> Fix for Script - **How to Identify a Scheduled SSRS report in SQL Agent Jobs**
```
SELECT
     b.name AS JobName
   , e.name
   , e.path
   , d.description
   , a.SubscriptionID
   , laststatus
   , eventtype
   , LastRunTime
   , date_created
   , date_modified
FROM 
   ReportServer.dbo.ReportSchedule a 
   JOIN msdb.dbo.sysjobs b ON convert(nvarchar(36),a.ScheduleID) = b.name
   JOIN ReportServer.dbo.ReportSchedule c ON b.name = convert(nvarchar(36),c.ScheduleID)
   JOIN ReportServer.dbo.Subscriptions d  ON convert(nvarchar(36),c.SubscriptionID) = convert(nvarchar(36),d.SubscriptionID)
   JOIN ReportServer.dbo.Catalog e ON d.report_oid = e.itemid
   ```
**Temp Table To Report Fragmented Indexes**
```
    -- If the temporary table already exists, drop it
     IF OBJECT_ID('tempdb..#FragTables') IS NOT NULL
         DROP TABLE #FragTables
     ;
    -- Create the temporary table
     CREATE TABLE #FragTables (
           SRVName NVARCHAR(255)
         , DBName NVARCHAR(255)
         , TableName NVARCHAR(255)
         , SchemaName NVARCHAR(255)
         , IndexName NVARCHAR(255)
         , IndexTypeDesc NVARCHAR(255)
         , AvgFragment TINYINT
         , PageCount int)
     ;
     -- Insert fragmented indices into the temporary table
     EXECUTE sp_MSforeachdb
         'INSERT INTO #FragTables (
               SRVName  
             , DBName
             , TableName
             , SchemaName
             , IndexName
             , IndexTypeDesc
             , AvgFragment
             , PageCount)   
         SELECT
             ( select upper(@@SERVERNAME)
	) AS SRVName 
             ,''?'' AS DBName
             , t.Name AS TableName
             , sc.Name AS SchemaName
             , i.name AS IndexName
             , s.index_type_desc
             , s.avg_fragmentation_in_percent
             , s.page_count
         FROM
             ?.sys.dm_db_index_physical_stats(
                 DB_ID(''?'')
                 , NULL
                 , NULL
                 , NULL
                 , ''Limited'') AS s
             INNER JOIN ?.sys.indexes AS i
                 ON s.Object_Id = i.Object_id
                 AND s.Index_id = i.Index_id
             INNER JOIN ?.sys.tables AS t
                 ON i.Object_id = t.Object_Id
             INNER JOIN ?.sys.schemas AS sc
                 ON t.schema_id = sc.SCHEMA_ID
         WHERE
             s.avg_fragmentation_in_percent > 30
             AND t.TYPE = ''U''
             and s.index_type_desc in (''CLUSTERED INDEX'', ''NONCLUSTERED INDEX'')
             and s.page_count >= 1000
             and ''?'' not in (''msdb'', ''master'', ''tempdb'', ''model'')
             and ''?'' not like ''%TEST%''
             and ''?'' not like ''DEV_%''
             and ''?'' not like ''%DEMO%''
             and ''?'' not like ''AdventureWorks%''
             and ''?'' not like ''u_%''
        ;'
     ;

INSERT INTO DBA_Monitoring.dbo.FragmentedIndexTable 
(
 SRVName
, DBName
, TableName
, SchemaName
, IndexName
, IndexTypeDesc
, AvgFragment
, PageCount
)    
SELECT
 SRVName
, DBName
, TableName
, SchemaName
, IndexName
, IndexTypeDesc
, AvgFragment
, PageCount
FROM #FragTables
```

**Generate XML Email from multiple SQL Instances**
```
DECLARE @xml NVARCHAR(MAX)
DECLARE @body NVARCHAR(MAX)

SET @xml = 
CAST
(
(
SELECT 
	[SRVName] AS 'td'
,'',
	[DBName] AS 'td'
,'',
	[TableName] AS 'td'
,'',
	[SchemaName] AS 'td'
,'',
	[IndexName] AS 'td'
,'',
	[IndexTypeDesc] AS 'td'
,'',
	[AvgFragment] AS 'td'
,'',	
	[PageCount] AS 'td'
 FROM
(
SELECT 
	[SRVName]
,	[DBName]
,	[TableName]
,	[SchemaName]
,	[IndexName]
,	[IndexTypeDesc]
,	[AvgFragment]
,	[PageCount] 
FROM srvsql8.dba_monitoring.dbo.FragmentedIndexTable
union
SELECT 
	[SRVName]
,	[DBName]
,	[TableName]
,	[SchemaName]
,	[IndexName]
,	[IndexTypeDesc]
,	[AvgFragment]
,	[PageCount]
FROM srvbi.dba_monitoring.dbo.FragmentedIndexTable
union
SELECT 
	[SRVName]
,	[DBName]
,	[TableName]
,	[SchemaName]
,	[IndexName]
,	[IndexTypeDesc]
,	[AvgFragment]
,	[PageCount]
FROM srvsqlnhc.dba_monitoring.dbo.FragmentedIndexTable
union
SELECT 
	[SRVName]
,	[DBName]
,	[TableName]
,	[SchemaName]
,	[IndexName]
,	[IndexTypeDesc]
,	[AvgFragment]
,	[PageCount]
FROM srvsqlpubsafe2.dba_monitoring.dbo.FragmentedIndexTable
) A
order by SRVName, DBName, AvgFragment desc
FOR XML PATH('tr')
,	ELEMENTS
) AS NVARCHAR(MAX)
)
SET 
	@body= 
	'
	<html>
	<body>
	<H3>Fragmented Index Report</H3>
	<p>This report provides an audit of 
	heavily fragmented indexes above 30 percent fragmentation. 
	To improve query performance rebuild 
	for the following indexes is recommended:</p>
	<table border = 1>
	<tr>
	<th>SRVName</th>
	<th>DBName</th>
	<th>TableName</th>
	<th>SchemaName</th>
	<th>IndexName</th>
	<th>IndexTypeDesc</th>
	<th>AvgFragment</th>
	<th>PageCount</th>
	'
SET @body =@body + @xml + 
	'
	</table>
	</body>
	</html>
	'
EXEC msdb.dbo.sp_send_dbmail 
@profile_name='DBMonitoring',
@body = @body,
@body_format= 'HTML',
@recipients='D_INFO_SQL_Alerts@nhcgov.com',
@subject='Quarterly Fragmented Index Audit'
```
