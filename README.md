# Microsoft SQL Server - Cheatsheet 
Useful SQL Server scripts and snippets

## Useful Tools

* [Brent Ozar's First Responder Kit](https://github.com/BrentOzarULTD/SQL-Server-First-Responder-Kit)
* [Adam Mechanic's sp_WhoIsActive](https://github.com/amachanic/sp_whoisactive/releases)
* [Ola Hallengren Scripts](https://ola.hallengren.com/)
* [RedGate SQL Search](https://www.red-gate.com/dynamic/products/sql-development/sql-search/download)
* [Statistics Parser](http://statisticsparser.com/)
* [Brent Ozar's Paste the Plan](https://www.brentozar.com/pastetheplan/)
* [dbatools (Instance Migration)](https://dbatools.io/)
* [SentryOne (Mostly paid)](https://www.sentryone.com/)
* [RedGate (Mostly paid)](https://www.red-gate.com)
* SQL Profiler
* Extended Events
* Visual Studio Schema Compare
* Visual Studio Data Compare
* Query Store

## Blogs & Resources
* [SQL Server Central](https://www.sqlservercentral.com/)
* [Brent Ozar's Blog](https://www.brentozar.com/blog/)
* [Brent Ozar's YouTube](https://www.youtube.com/user/BrentOzar)
* [Kendra Little's YouTube](https://www.youtube.com/channel/UCrJ8WLrVoKxL94mKv2akxTA/videos)
* [SQLskills Resources](https://www.sqlskills.com/sql-server-resources/)
* [StackOverflow Database Backups](https://www.brentozar.com/archive/2015/10/how-to-download-the-stack-overflow-database-via-bittorrent/)
* [DBA StackExchange](https://dba.stackexchange.com/?tags=sql-server)
* [RedGate's Simple Talk Blog](https://www.red-gate.com/simple-talk/)
* [SentryOne Blog](https://www.sentryone.com/blog)
* [...More blogs](https://www.sqlshack.com/sql-server-blogs/)

## Database specific queries

<details>
 <summary>Table row counts & disk usage</summary>
 
  ```sql
  SELECT 
   t.NAME AS TableName,
   i.name AS indexName,
   SUM(p.rows) AS RowCounts,
   SUM(a.total_pages) AS TotalPages, 
   SUM(a.used_pages) AS UsedPages, 
   SUM(a.data_pages) AS DataPages,
   (SUM(a.total_pages) * 8) / 1024 AS TotalSpaceMB, 
   (SUM(a.used_pages) * 8) / 1024 AS UsedSpaceMB, 
   (SUM(a.data_pages) * 8) / 1024 AS DataSpaceMB
  FROM 
   sys.tables t
  INNER JOIN  
   sys.indexes i ON t.OBJECT_ID = i.object_id
  INNER JOIN 
   sys.partitions p ON i.object_id = p.OBJECT_ID AND i.index_id = p.index_id
  INNER JOIN 
   sys.allocation_units a ON p.partition_id = a.container_id
  WHERE 
   t.NAME NOT LIKE 'dt%' AND
   i.OBJECT_ID > 255 AND  
   i.index_id <= 1
  GROUP BY 
   t.NAME, i.object_id, i.index_id, i.name 
  ORDER BY 
   OBJECT_NAME(i.object_id) 
  ```
</details>

## Server specific queries

<details>
 <summary>Total disk usage for server</summary>
 
 ```sql
 SELECT CONVERT(DECIMAL(10,2),(SUM(size * 8.00) / 1024.00 / 1024.00)) As UsedSpace
 FROM master.sys.master_files
 ```
</details>
