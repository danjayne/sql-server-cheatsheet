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
* [PsExec (run SSMS as NT AUTHORITY\SYSTEM)](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec)
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

## Snippets
<details>
 <summary>SET STATISTICS</summary>
 
 ```sql
SET STATISTICS IO, TIME ON;
-- Query
SET STATISTICS IO, TIME OFF;
 ```
</details>



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

<details>
 <summary>sp_WhoIsActive</summary>
 
 ```sql
	EXEC [dbo].[sp_WhoIsActive] @get_full_inner_text = 1, @get_outer_command = 1, @get_plans = 1
 ```
</details>

<details>
 <summary>sp_BlitzFirst</summary>
 
 ```sql
	EXEC [dbo].[sp_BlitzFirst] @Seconds = 10, @ExpertMode = 1, @ShowSleepingSPIDS = 1
 ```
</details>

<details>
 <summary>sp_Blitz</summary>
 
 ```sql
	EXEC sp_Blitz
 ```
</details>

<details>
 <summary>sp_BlitzCache</summary>
 
  ```sql
	EXEC sp_BlitzCache @Top = 10,  @SortOrder = 'cpu', @ExpertMode = 1, @MinimumExecutionCount = 2
 ```
  ```sql
	EXEC sp_BlitzCache @Top = 10,  @SortOrder = 'executions', @ExpertMode = 1, @MinimumExecutionCount = 2
 ```
 ```sql
	EXEC sp_BlitzCache @Top = 100,  @SortOrder = 'avg reads', @ExpertMode = 1, @MinimumExecutionCount = 10
 ```
 ```sql
	EXEC sp_BlitzCache @Top = 10,  @SortOrder = 'recent compilations', @ExpertMode = 1, @MinimumExecutionCount = 2
 ```
</details>

<details>
 <summary>sp_BlitzIndex</summary>
 
 ```sql
	EXEC sp_BlitzIndex
 ```
</details>

## Server specific queries

<details>
 <summary>DBCC SQLPERF</summary>
 
 ```sql
 DBCC SQLPERF (LOGSPACE);
 ```
 
 ```sql
DBCC SQLPERF("sys.dm_os_latch_stats" , CLEAR);
DBCC SQLPERF("sys.dm_os_wait_stats" , CLEAR);
GO
 ```
</details>

<details>
 <summary>DBCC SHOW_STATISTICS</summary>
 
 ```sql
DBCC SHOW_STATISTICS('dbo.TableName', 'PK_TableName_Id');
 ```
</details>

<details>
 <summary>Total disk usage for server</summary>
 
 ```sql
 SELECT CONVERT(DECIMAL(10,2),(SUM(size * 8.00) / 1024.00 / 1024.00)) As UsedSpace
 FROM master.sys.master_files
 ```
</details>

<details>
 <summary>Ola's IndexOptimize Script</summary>
 
 ```sql
EXECUTE dbo.IndexOptimize
@Databases = 'USER_DATABASES',
@FragmentationLow = NULL,
@FragmentationMedium = 'INDEX_REORGANIZE,INDEX_REBUILD_ONLINE,INDEX_REBUILD_OFFLINE',
@FragmentationHigh = 'INDEX_REBUILD_ONLINE,INDEX_REBUILD_OFFLINE',
@FragmentationLevel1 = 5,
@FragmentationLevel2 = 30,
@UpdateStatistics = 'ALL',
@OnlyModifiedStatistics = 'Y'
 ```
 
 ```sql
EXECUTE dbo.IndexOptimize
@Databases = 'USER_DATABASES',
@FragmentationLow = NULL,
@FragmentationMedium = NULL,
@FragmentationHigh = NULL,
@UpdateStatistics = 'ALL'
 ```
</details>

<details>
 <summary>Ola's DatabaseIntegrityCheck Script</summary>
 
 ```sql
EXECUTE dbo.DatabaseIntegrityCheck
@Databases = 'USER_DATABASES',
@CheckCommands = 'CHECKDB'
 ```
</details>

<details>
 <summary>Ola's DatabaseBackup Script</summary>
 
```sql
EXECUTE dbo.DatabaseBackup
@Databases = 'USER_DATABASES',
@Directory = 'C:\Backup',
@MirrorDirectory = 'D:\Backup',
@BackupType = 'FULL',
@Compress = 'Y',
@Verify = 'Y',
@CleanupTime = 24,
@MirrorCleanupTime = 48	
```

```sql
EXECUTE dbo.DatabaseBackup
@Databases = 'SYSTEM_DATABASES',
@Directory = 'C:\Backup',
@BackupType = 'FULL',
@Verify = 'Y',
@Compress = 'Y',
@CheckSum = 'Y',
@CleanupTime = 24
```

```sql
EXECUTE dbo.DatabaseBackup
@Databases = 'USER_DATABASES',
@Directory = 'C:\Backup',
@BackupType = 'LOG',
@Verify = 'Y',
@Compress = 'Y',
@CheckSum = 'Y',
@CleanupTime = 1
```

```sql
EXECUTE dbo.DatabaseBackup
@Databases = 'USER_DATABASES',
@Directory = 'C:\Backup',
@BackupType = 'DIFF',
@Verify = 'Y',
@Compress = 'Y',
@CheckSum = 'Y',
@CleanupTime = 6
```
</details>

<details>
 <summary>Clear proc cache (Azure SQL)</summary>
 
 ```sql
ALTER DATABASE SCOPED CONFIGURATION CLEAR PROCEDURE_CACHE ;  
 ```
 
```sql
ALTER DATABASE SCOPED CONFIGURATION CLEAR PROCEDURE_CACHE (0x0...);  
 ```
</details>
