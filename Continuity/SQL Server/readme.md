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
 ---  

