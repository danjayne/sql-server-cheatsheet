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
* [RML Utilities (includes ostress)](https://www.microsoft.com/en-us/download/details.aspx?id=4511)
* [dbForge SQL Complete](https://www.devart.com/dbforge/sql/sqlcomplete/)
* [Apex SQL Tools](https://www.apexsql.com/Download.aspx)
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
* [Useful T-SQL queries and scripts](https://www.sqlservercentral.com/blogs/useful-t-sql-queries-and-scripts-to-work-in-sql-server)
* [Reddit - "A big list of sql server queries and commands"](https://www.reddit.com/r/learnprogramming/comments/7fm6k6/sql_a_big_list_of_sql_server_queries_and_commands/)
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
	<summary>Track progress of Index creation</summary>
	
```sql
-- Index creation query must have SET STATISTICS PROFILE ON;
-- Grab @SPID value from sp_WhoIsActive

DECLARE @SPID INT = 51;

;WITH agg AS
(
		 SELECT SUM(qp.[row_count]) AS [RowsProcessed],
						SUM(qp.[estimate_row_count]) AS [TotalRows],
						MAX(qp.last_active_time) - MIN(qp.first_active_time) AS [ElapsedMS],
						MAX(IIF(qp.[close_time] = 0 AND qp.[first_row_time] > 0,
										[physical_operator_name],
										N'<Transition>')) AS [CurrentStep]
		 FROM sys.dm_exec_query_profiles qp
		 WHERE qp.[physical_operator_name] IN (N'Table Scan', N'Clustered Index Scan',
																					 N'Index Scan',  N'Sort')
		 AND   qp.[session_id] = @SPID
), comp AS
(
		 SELECT *,
						([TotalRows] - [RowsProcessed]) AS [RowsLeft],
						([ElapsedMS] / 1000.0) AS [ElapsedSeconds]
		 FROM   agg
)
SELECT [CurrentStep],
			 [TotalRows],
			 [RowsProcessed],
			 [RowsLeft],
			 CONVERT(DECIMAL(5, 2),
							 (([RowsProcessed] * 1.0) / [TotalRows]) * 100) AS [PercentComplete],
			 [ElapsedSeconds],
			 (([ElapsedSeconds] / [RowsProcessed]) * [RowsLeft]) AS [EstimatedSecondsLeft],
			 DATEADD(SECOND,
							 (([ElapsedSeconds] / [RowsProcessed]) * [RowsLeft]),
							 GETDATE()) AS [EstimatedCompletionTime]
FROM   comp;

```
</details>

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
EXECUTE [dbo].[sp_WhoIsActive] @get_full_inner_text = 1, @get_outer_command = 1, @get_plans = 1
 ```
</details>

<details>
 <summary>sp_BlitzFirst</summary>
 
 ```sql
EXECUTE [dbo].[sp_BlitzFirst] @Seconds = 10, @ExpertMode = 1, @ShowSleepingSPIDS = 1
 ```
</details>

<details>
 <summary>sp_Blitz</summary>
 
 ```sql
EXECUTE sp_Blitz
 ```
</details>

<details>
 <summary>sp_BlitzCache</summary>
 
```sql
EXECUTE sp_BlitzCache @Top = 10,  @SortOrder = 'cpu', @ExpertMode = 1, @MinimumExecutionCount = 2
```
```sql
EXECUTE sp_BlitzCache @Top = 10,  @SortOrder = 'executions', @ExpertMode = 1, @MinimumExecutionCount = 2
```
```sql
EXECUTE sp_BlitzCache @Top = 100,  @SortOrder = 'avg reads', @ExpertMode = 1, @MinimumExecutionCount = 10
```
```sql
EXECUTE sp_BlitzCache @Top = 10,  @SortOrder = 'recent compilations', @ExpertMode = 1, @MinimumExecutionCount = 2
```
</details>

<details>
 <summary>sp_BlitzIndex</summary>
 
 ```sql
EXECUTE sp_BlitzIndex
 ```
</details>

<details>
	<summary>Database Backups for Previous Week</summary>
	
```sql
--------------------------------------------------------------------------------- 
--Database Backups for all databases For Previous Week 
--------------------------------------------------------------------------------- 
SELECT 
CONVERT(CHAR(100), SERVERPROPERTY('Servername')) AS Server, 
msdb.dbo.backupset.database_name, 
msdb.dbo.backupset.backup_start_date, 
msdb.dbo.backupset.backup_finish_date, 
msdb.dbo.backupset.expiration_date, 
CASE msdb..backupset.type 
WHEN 'D' THEN 'Database' 
WHEN 'L' THEN 'Log' 
END AS backup_type, 
msdb.dbo.backupset.backup_size, 
--convert(decimal(18,3),(sum(msdb.dbo.backupset.backup_size))/1024/1024/1024) as SizeinGB,
msdb.dbo.backupmediafamily.logical_device_name, 
msdb.dbo.backupmediafamily.physical_device_name, 
msdb.dbo.backupset.name AS backupset_name, 
msdb.dbo.backupset.description 
FROM msdb.dbo.backupmediafamily 
INNER JOIN msdb.dbo.backupset ON msdb.dbo.backupmediafamily.media_set_id = msdb.dbo.backupset.media_set_id 
WHERE (CONVERT(datetime, msdb.dbo.backupset.backup_start_date, 102) >= GETDATE() - 7) 
AND msdb.dbo.backupset.database_name = 'DatabaseName' AND msdb..backupset.type = 'D'
ORDER BY 
msdb.dbo.backupset.database_name, 
msdb.dbo.backupset.backup_finish_date 
```
</details>

<details>
	<summary>Database physical size vs used space</summary>
	
```sql
IF OBJECT_ID('tempdb.dbo.#space') IS NOT NULL
		DROP TABLE #space
CREATE TABLE #space (
			database_id INT PRIMARY KEY
		, data_used_size DECIMAL(18,2)
		, log_used_size DECIMAL(18,2)
)
DECLARE @SQL NVARCHAR(MAX)
SELECT @SQL = STUFF((
		SELECT '
		USE [' + d.name + ']
		INSERT INTO #space (database_id, data_used_size, log_used_size)
		SELECT
					DB_ID()
				, SUM(CASE WHEN [type] = 0 THEN space_used END)
				, SUM(CASE WHEN [type] = 1 THEN space_used END)
		FROM (
				SELECT s.[type], space_used = SUM(FILEPROPERTY(s.name, ''SpaceUsed'') * 8. / 1024)
				FROM sys.database_files s
				GROUP BY s.[type]
		) t;'
		FROM sys.databases d
		WHERE d.[state] = 0
		FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 2, '')
EXECUTE sys.sp_executesql @SQL
SELECT
			d.database_id
		, d.name
		, d.state_desc
		, d.recovery_model_desc
		, t.total_size
		, t.data_size
		, s.data_used_size
		, t.log_size
		, s.log_used_size
		, bu.full_last_date
		, bu.full_size
		, bu.log_last_date
		, bu.log_size
FROM (
		SELECT
					database_id
				, log_size = CAST(SUM(CASE WHEN [type] = 1 THEN size END) * 8. / 1024 AS DECIMAL(18,2))
				, data_size = CAST(SUM(CASE WHEN [type] = 0 THEN size END) * 8. / 1024 AS DECIMAL(18,2))
				, total_size = CAST(SUM(size) * 8. / 1024 AS DECIMAL(18,2))
		FROM sys.master_files
		GROUP BY database_id
) t
JOIN sys.databases d ON d.database_id = t.database_id
LEFT JOIN #space s ON d.database_id = s.database_id
LEFT JOIN (
		SELECT
					database_name
				, full_last_date = MAX(CASE WHEN [type] = 'D' THEN backup_finish_date END)
				, full_size = MAX(CASE WHEN [type] = 'D' THEN backup_size END)
				, log_last_date = MAX(CASE WHEN [type] = 'L' THEN backup_finish_date END)
				, log_size = MAX(CASE WHEN [type] = 'L' THEN backup_size END)
		FROM (
				SELECT
							s.database_name
						, s.[type]
						, s.backup_finish_date
						, backup_size =
												CAST(CASE WHEN s.backup_size = s.compressed_backup_size
																		THEN s.backup_size
																		ELSE s.compressed_backup_size
												END / 1048576.0 AS DECIMAL(18,2))
						, RowNum = ROW_NUMBER() OVER (PARTITION BY s.database_name, s.[type] ORDER BY s.backup_finish_date DESC)
				FROM msdb.dbo.backupset s
				WHERE s.[type] IN ('D', 'L')
		) f
		WHERE f.RowNum = 1
		GROUP BY f.database_name
) bu ON d.name = bu.database_name
ORDER BY t.total_size DESC
```
</details>

<details>
	<summary>Date statistics were last updated</summary>

```sql
SELECT ss.name AS SchemaName
, obj.name AS TableName
, sp.last_updated
, stat.stats_id
, stat.name AS StatiscticsName
, stat.filter_definition
, sp.[rows]
, sp.rows_sampled
, sp.steps
, sp.unfiltered_rows
, sp.modification_counter
FROM sys.objects AS obj
INNER JOIN sys.schemas ss ON obj.schema_id = ss.schema_id
INNER JOIN sys.stats stat ON stat.object_id = obj.object_id
CROSS APPLY sys.dm_db_stats_properties(stat.object_id, stat.stats_id) AS sp
WHERE obj.is_ms_shipped = 0
ORDER BY ss.name, obj.name, sp.last_updated desc
```
</details>

<details>
	<summary>Date statistics were last updated (alt)</summary>
	
```sql
SELECT s.name AS Stats, o.name AS TableName, STATS_DATE(s.object_id, s.stats_id) AS LastStatsUpdate
FROM sys.stats s
JOIN sys.objects o ON s.object_id = o.object_id
WHERE
	1 = 1
	AND s.auto_created = 0 AND left(s.name,4)!='_WA_' -- Comment this out to just look at index stats
	AND is_ms_shipped = 0 -- Comment this out to include system table stats
ORDER BY LastStatsUpdate
```
</details>

<details>
 <summary>DBCC SHOW_STATISTICS</summary>
 
```sql
DBCC SHOW_STATISTICS('dbo.TableName', 'PK_TableName_Id');
```
</details>


<details>
 <summary>Add Table and Column comments (~Azure SQL~)</summary>
 
```sql
declare @CurrentUser sysname
select @CurrentUser = user_name()
execute sp_addextendedproperty 'MS_Description', 
   'This is my table comment',
   'user', @CurrentUser, 'table', 'TABLE_1'
go

declare @CurrentUser sysname
select @CurrentUser = user_name()
execute sp_addextendedproperty 'MS_Description', 
   'This is the primary key comment',
   'user', @CurrentUser, 'table', 'TABLE_1', 'column', 'ID'
go

declare @CurrentUser sysname
select @CurrentUser = user_name()
execute sp_addextendedproperty 'MS_Description', 
   'This is column one comment',
   'user', @CurrentUser, 'table', 'TABLE_1', 'column', 'COLUMN_1'
go

declare @CurrentUser sysname
select @CurrentUser = user_name()
execute sp_addextendedproperty 'MS_Description', 
   'This is column 2 comment',
   'user', @CurrentUser, 'table', 'TABLE_1', 'column', 'COLUMN_2'
go
```
</details>

<details>
 <summary>Parameter list for Stored Procedure</summary>
 
```sql
SELECT 'Parameter_name' = name,
	'Type' = TYPE_NAME(user_type_id),
	'Length' = max_length,
	'Prec' = CASE
				WHEN TYPE_NAME(system_type_id) = 'uniqueidentifier'
				THEN precision
				ELSE OdbcPrec(system_type_id, max_length, precision)
			END,
	'Scale' = OdbcScale(system_type_id, scale),
	'Param_order' = parameter_id,
	'Collation' = CONVERT(SYSNAME,
							CASE
								WHEN system_type_id IN(35, 99, 167, 175, 231, 239)
								THEN SERVERPROPERTY('collation')
							END)
FROM sys.parameters
WHERE object_id = OBJECT_ID('dbo.TableName');
```
</details>

<details>
 <summary>All missing dependancies</summary>
 
```sql
SELECT DISTINCT 'DROP PROCEDURE IF EXISTS ' +  OBJECT_SCHEMA_NAME(referencing_id) + '.' + 
		OBJECT_NAME(referencing_id) AS [referencer],
		referenced_entity_name AS [referenced]
FROM sys.sql_expression_dependencies
WHERE is_ambiguous = 0
		AND OBJECT_ID(ISNULL(referenced_schema_name, 'dbo') + '.' + referenced_entity_name) IS NULL
		AND OBJECT_ID(ISNULL(referenced_schema_name, OBJECT_SCHEMA_NAME(referencing_id)) + '.' + referenced_entity_name) IS NULL
		AND referenced_entity_name NOT IN (SELECT Name FROM sys.types WHERE is_user_defined = 1) -- avoid type false positives
		AND referenced_entity_name not in ('deleted', 'inserted') -- avoid trigger false positives
		AND referenced_database_name is null
--ORDER BY OBJECT_NAME(referencing_id), referenced_entity_name
```
</details>

<details>
 <summary>Tell me where it hurts</summary>
 
```sql
-- Last updated February 26, 2019
WITH [Waits] AS
    (SELECT
        [wait_type],
        [wait_time_ms] / 1000.0 AS [WaitS],
        ([wait_time_ms] - [signal_wait_time_ms]) / 1000.0 AS [ResourceS],
        [signal_wait_time_ms] / 1000.0 AS [SignalS],
        [waiting_tasks_count] AS [WaitCount],
        100.0 * [wait_time_ms] / SUM ([wait_time_ms]) OVER() AS [Percentage],
        ROW_NUMBER() OVER(ORDER BY [wait_time_ms] DESC) AS [RowNum]
    FROM sys.dm_os_wait_stats
    WHERE [wait_type] NOT IN (
        -- These wait types are almost 100% never a problem and so they are
        -- filtered out to avoid them skewing the results. Click on the URL
        -- for more information.
        N'BROKER_EVENTHANDLER', -- https://www.sqlskills.com/help/waits/BROKER_EVENTHANDLER
        N'BROKER_RECEIVE_WAITFOR', -- https://www.sqlskills.com/help/waits/BROKER_RECEIVE_WAITFOR
        N'BROKER_TASK_STOP', -- https://www.sqlskills.com/help/waits/BROKER_TASK_STOP
        N'BROKER_TO_FLUSH', -- https://www.sqlskills.com/help/waits/BROKER_TO_FLUSH
        N'BROKER_TRANSMITTER', -- https://www.sqlskills.com/help/waits/BROKER_TRANSMITTER
        N'CHECKPOINT_QUEUE', -- https://www.sqlskills.com/help/waits/CHECKPOINT_QUEUE
        N'CHKPT', -- https://www.sqlskills.com/help/waits/CHKPT
        N'CLR_AUTO_EVENT', -- https://www.sqlskills.com/help/waits/CLR_AUTO_EVENT
        N'CLR_MANUAL_EVENT', -- https://www.sqlskills.com/help/waits/CLR_MANUAL_EVENT
        N'CLR_SEMAPHORE', -- https://www.sqlskills.com/help/waits/CLR_SEMAPHORE
        N'CXCONSUMER', -- https://www.sqlskills.com/help/waits/CXCONSUMER
 
        -- Maybe comment these four out if you have mirroring issues
        N'DBMIRROR_DBM_EVENT', -- https://www.sqlskills.com/help/waits/DBMIRROR_DBM_EVENT
        N'DBMIRROR_EVENTS_QUEUE', -- https://www.sqlskills.com/help/waits/DBMIRROR_EVENTS_QUEUE
        N'DBMIRROR_WORKER_QUEUE', -- https://www.sqlskills.com/help/waits/DBMIRROR_WORKER_QUEUE
        N'DBMIRRORING_CMD', -- https://www.sqlskills.com/help/waits/DBMIRRORING_CMD
 
        N'DIRTY_PAGE_POLL', -- https://www.sqlskills.com/help/waits/DIRTY_PAGE_POLL
        N'DISPATCHER_QUEUE_SEMAPHORE', -- https://www.sqlskills.com/help/waits/DISPATCHER_QUEUE_SEMAPHORE
        N'EXECSYNC', -- https://www.sqlskills.com/help/waits/EXECSYNC
        N'FSAGENT', -- https://www.sqlskills.com/help/waits/FSAGENT
        N'FT_IFTS_SCHEDULER_IDLE_WAIT', -- https://www.sqlskills.com/help/waits/FT_IFTS_SCHEDULER_IDLE_WAIT
        N'FT_IFTSHC_MUTEX', -- https://www.sqlskills.com/help/waits/FT_IFTSHC_MUTEX
 
        -- Maybe comment these six out if you have AG issues
        N'HADR_CLUSAPI_CALL', -- https://www.sqlskills.com/help/waits/HADR_CLUSAPI_CALL
        N'HADR_FILESTREAM_IOMGR_IOCOMPLETION', -- https://www.sqlskills.com/help/waits/HADR_FILESTREAM_IOMGR_IOCOMPLETION
        N'HADR_LOGCAPTURE_WAIT', -- https://www.sqlskills.com/help/waits/HADR_LOGCAPTURE_WAIT
        N'HADR_NOTIFICATION_DEQUEUE', -- https://www.sqlskills.com/help/waits/HADR_NOTIFICATION_DEQUEUE
        N'HADR_TIMER_TASK', -- https://www.sqlskills.com/help/waits/HADR_TIMER_TASK
        N'HADR_WORK_QUEUE', -- https://www.sqlskills.com/help/waits/HADR_WORK_QUEUE
 
        N'KSOURCE_WAKEUP', -- https://www.sqlskills.com/help/waits/KSOURCE_WAKEUP
        N'LAZYWRITER_SLEEP', -- https://www.sqlskills.com/help/waits/LAZYWRITER_SLEEP
        N'LOGMGR_QUEUE', -- https://www.sqlskills.com/help/waits/LOGMGR_QUEUE
        N'MEMORY_ALLOCATION_EXT', -- https://www.sqlskills.com/help/waits/MEMORY_ALLOCATION_EXT
        N'ONDEMAND_TASK_QUEUE', -- https://www.sqlskills.com/help/waits/ONDEMAND_TASK_QUEUE
        N'PARALLEL_REDO_DRAIN_WORKER', -- https://www.sqlskills.com/help/waits/PARALLEL_REDO_DRAIN_WORKER
        N'PARALLEL_REDO_LOG_CACHE', -- https://www.sqlskills.com/help/waits/PARALLEL_REDO_LOG_CACHE
        N'PARALLEL_REDO_TRAN_LIST', -- https://www.sqlskills.com/help/waits/PARALLEL_REDO_TRAN_LIST
        N'PARALLEL_REDO_WORKER_SYNC', -- https://www.sqlskills.com/help/waits/PARALLEL_REDO_WORKER_SYNC
        N'PARALLEL_REDO_WORKER_WAIT_WORK', -- https://www.sqlskills.com/help/waits/PARALLEL_REDO_WORKER_WAIT_WORK
        N'PREEMPTIVE_OS_FLUSHFILEBUFFERS', -- https://www.sqlskills.com/help/waits/PREEMPTIVE_OS_FLUSHFILEBUFFERS 
        N'PREEMPTIVE_XE_GETTARGETSTATE', -- https://www.sqlskills.com/help/waits/PREEMPTIVE_XE_GETTARGETSTATE
        N'PWAIT_ALL_COMPONENTS_INITIALIZED', -- https://www.sqlskills.com/help/waits/PWAIT_ALL_COMPONENTS_INITIALIZED
        N'PWAIT_DIRECTLOGCONSUMER_GETNEXT', -- https://www.sqlskills.com/help/waits/PWAIT_DIRECTLOGCONSUMER_GETNEXT
        N'QDS_PERSIST_TASK_MAIN_LOOP_SLEEP', -- https://www.sqlskills.com/help/waits/QDS_PERSIST_TASK_MAIN_LOOP_SLEEP
        N'QDS_ASYNC_QUEUE', -- https://www.sqlskills.com/help/waits/QDS_ASYNC_QUEUE
        N'QDS_CLEANUP_STALE_QUERIES_TASK_MAIN_LOOP_SLEEP',
            -- https://www.sqlskills.com/help/waits/QDS_CLEANUP_STALE_QUERIES_TASK_MAIN_LOOP_SLEEP
        N'QDS_SHUTDOWN_QUEUE', -- https://www.sqlskills.com/help/waits/QDS_SHUTDOWN_QUEUE
        N'REDO_THREAD_PENDING_WORK', -- https://www.sqlskills.com/help/waits/REDO_THREAD_PENDING_WORK
        N'REQUEST_FOR_DEADLOCK_SEARCH', -- https://www.sqlskills.com/help/waits/REQUEST_FOR_DEADLOCK_SEARCH
        N'RESOURCE_QUEUE', -- https://www.sqlskills.com/help/waits/RESOURCE_QUEUE
        N'SERVER_IDLE_CHECK', -- https://www.sqlskills.com/help/waits/SERVER_IDLE_CHECK
        N'SLEEP_BPOOL_FLUSH', -- https://www.sqlskills.com/help/waits/SLEEP_BPOOL_FLUSH
        N'SLEEP_DBSTARTUP', -- https://www.sqlskills.com/help/waits/SLEEP_DBSTARTUP
        N'SLEEP_DCOMSTARTUP', -- https://www.sqlskills.com/help/waits/SLEEP_DCOMSTARTUP
        N'SLEEP_MASTERDBREADY', -- https://www.sqlskills.com/help/waits/SLEEP_MASTERDBREADY
        N'SLEEP_MASTERMDREADY', -- https://www.sqlskills.com/help/waits/SLEEP_MASTERMDREADY
        N'SLEEP_MASTERUPGRADED', -- https://www.sqlskills.com/help/waits/SLEEP_MASTERUPGRADED
        N'SLEEP_MSDBSTARTUP', -- https://www.sqlskills.com/help/waits/SLEEP_MSDBSTARTUP
        N'SLEEP_SYSTEMTASK', -- https://www.sqlskills.com/help/waits/SLEEP_SYSTEMTASK
        N'SLEEP_TASK', -- https://www.sqlskills.com/help/waits/SLEEP_TASK
        N'SLEEP_TEMPDBSTARTUP', -- https://www.sqlskills.com/help/waits/SLEEP_TEMPDBSTARTUP
        N'SNI_HTTP_ACCEPT', -- https://www.sqlskills.com/help/waits/SNI_HTTP_ACCEPT
        N'SOS_WORK_DISPATCHER', -- https://www.sqlskills.com/help/waits/SOS_WORK_DISPATCHER
        N'SP_SERVER_DIAGNOSTICS_SLEEP', -- https://www.sqlskills.com/help/waits/SP_SERVER_DIAGNOSTICS_SLEEP
        N'SQLTRACE_BUFFER_FLUSH', -- https://www.sqlskills.com/help/waits/SQLTRACE_BUFFER_FLUSH
        N'SQLTRACE_INCREMENTAL_FLUSH_SLEEP', -- https://www.sqlskills.com/help/waits/SQLTRACE_INCREMENTAL_FLUSH_SLEEP
        N'SQLTRACE_WAIT_ENTRIES', -- https://www.sqlskills.com/help/waits/SQLTRACE_WAIT_ENTRIES
        N'VDI_CLIENT_OTHER', -- https://www.sqlskills.com/help/waits/VDI_CLIENT_OTHER
        N'WAIT_FOR_RESULTS', -- https://www.sqlskills.com/help/waits/WAIT_FOR_RESULTS
        N'WAITFOR', -- https://www.sqlskills.com/help/waits/WAITFOR
        N'WAITFOR_TASKSHUTDOWN', -- https://www.sqlskills.com/help/waits/WAITFOR_TASKSHUTDOWN
        N'WAIT_XTP_RECOVERY', -- https://www.sqlskills.com/help/waits/WAIT_XTP_RECOVERY
        N'WAIT_XTP_HOST_WAIT', -- https://www.sqlskills.com/help/waits/WAIT_XTP_HOST_WAIT
        N'WAIT_XTP_OFFLINE_CKPT_NEW_LOG', -- https://www.sqlskills.com/help/waits/WAIT_XTP_OFFLINE_CKPT_NEW_LOG
        N'WAIT_XTP_CKPT_CLOSE', -- https://www.sqlskills.com/help/waits/WAIT_XTP_CKPT_CLOSE
        N'XE_DISPATCHER_JOIN', -- https://www.sqlskills.com/help/waits/XE_DISPATCHER_JOIN
        N'XE_DISPATCHER_WAIT', -- https://www.sqlskills.com/help/waits/XE_DISPATCHER_WAIT
        N'XE_TIMER_EVENT' -- https://www.sqlskills.com/help/waits/XE_TIMER_EVENT
        )
    AND [waiting_tasks_count] > 0
    )
SELECT
    MAX ([W1].[wait_type]) AS [WaitType],
    CAST (MAX ([W1].[WaitS]) AS DECIMAL (16,2)) AS [Wait_S],
    CAST (MAX ([W1].[ResourceS]) AS DECIMAL (16,2)) AS [Resource_S],
    CAST (MAX ([W1].[SignalS]) AS DECIMAL (16,2)) AS [Signal_S],
    MAX ([W1].[WaitCount]) AS [WaitCount],
    CAST (MAX ([W1].[Percentage]) AS DECIMAL (5,2)) AS [Percentage],
    CAST ((MAX ([W1].[WaitS]) / MAX ([W1].[WaitCount])) AS DECIMAL (16,4)) AS [AvgWait_S],
    CAST ((MAX ([W1].[ResourceS]) / MAX ([W1].[WaitCount])) AS DECIMAL (16,4)) AS [AvgRes_S],
    CAST ((MAX ([W1].[SignalS]) / MAX ([W1].[WaitCount])) AS DECIMAL (16,4)) AS [AvgSig_S],
    CAST ('https://www.sqlskills.com/help/waits/' + MAX ([W1].[wait_type]) as XML) AS [Help/Info URL]
FROM [Waits] AS [W1]
INNER JOIN [Waits] AS [W2] ON [W2].[RowNum] <= [W1].[RowNum]
GROUP BY [W1].[RowNum]
HAVING SUM ([W2].[Percentage]) - MAX( [W1].[Percentage] ) < 95; -- percentage threshold
GO
```
</details>

<details>
	<summary>DDL Trigger to track schema changes</summary>
	<p>

```sql
CREATE TABLE [dbo].[DDL_Change_Log](
	[EventDate] [datetime2](7) NOT NULL,
	[EventType] [nvarchar](100) NULL,
	[EventDDL] [nvarchar](max) NULL,
	[EventXML] [xml] NULL,
	[DatabaseName] [nvarchar](255) NULL,
	[SchemaName] [nvarchar](255) NULL,
	[ObjectName] [nvarchar](255) NULL,
	[HostName] [varchar](64) NULL,
	[IPAddress] [varchar](48) NULL,
	[ProgramName] [nvarchar](255) NULL,
	[LoginName] [nvarchar](255) NULL
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
ALTER TABLE [dbo].[DDL_Change_Log] ADD  DEFAULT (getutcdate()) FOR [EventDate]
GO
```
		
```sql
CREATE TRIGGER DDL_Change_Trigger
ON DATABASE
FOR CREATE_PROCEDURE, ALTER_PROCEDURE, DROP_PROCEDURE, 
	CREATE_TABLE, ALTER_TABLE, DROP_TABLE,
	CREATE_INDEX, ALTER_INDEX, DROP_INDEX,
	CREATE_FUNCTION, ALTER_FUNCTION, DROP_FUNCTION,
	CREATE_VIEW, ALTER_VIEW, DROP_VIEW
AS
BEGIN
	SET NOCOUNT ON;

	DECLARE @EventData XML = EVENTDATA();
	DECLARE @ip varchar(48) = CONVERT(varchar(48), CONNECTIONPROPERTY('client_net_address'));

	INSERT dbo.DDL_Change_Log
	(
		EventType,
		EventDDL,
		EventXML,
		DatabaseName,
		SchemaName,
		ObjectName,
		HostName,
		IPAddress,
		ProgramName,
		LoginName
	)
	SELECT
		@EventData.value('(/EVENT_INSTANCE/EventType)[1]',   'NVARCHAR(100)'), 
		@EventData.value('(/EVENT_INSTANCE/TSQLCommand)[1]', 'NVARCHAR(MAX)'),
		@EventData,
		DB_NAME(),
		@EventData.value('(/EVENT_INSTANCE/SchemaName)[1]',  'NVARCHAR(255)'), 
		@EventData.value('(/EVENT_INSTANCE/ObjectName)[1]',  'NVARCHAR(255)'),
		HOST_NAME(),
		@ip,
		PROGRAM_NAME(),
		SUSER_SNAME();
END
GO
ENABLE TRIGGER DDL_Change_Trigger ON DATABASE
GO
```
</p>
</details>

<details>
 <summary>List plan cache</summary>
 
```sql
CREATE OR ALTER FUNCTION [dbo].[SqlAndPlan](@handle varbinary(max))
RETURNS TABLE
AS
	RETURN SELECT sql.text, cp.usecounts, cp.cacheobjtype, cp.objtype, cp.size_in_bytes, qp.query_plan
		FROM
		sys.dm_exec_sql_text(@handle) AS SQL CROSS JOIN
		sys.dm_exec_query_plan(@handle) as qp
		JOIN sys.dm_exec_cached_plans AS cp
		ON cp.plan_handle = @handle;
GO

CREATE OR ALTER VIEW [dbo].[PlanCache]
AS
(
	SELECT sp.*, cp.plan_handle FROM sys.dm_exec_cached_plans AS cp
	CROSS APPLY SqlAndPlan(cp.plan_handle) AS sp
)
GO
```
 
```sql
SELECT * FROM [dbo].[PlanCache]
WHERE objtype = 'Proc'
ORDER BY usecounts
```
</details>

<details>
 <summary>Get current Isolation Level</summary>
 
```sql
SELECT CASE transaction_isolation_level 
WHEN 0 THEN 'Unspecified' 
WHEN 1 THEN 'ReadUncommitted' 
WHEN 2 THEN 'ReadCommitted' 
WHEN 3 THEN 'Repeatable' 
WHEN 4 THEN 'Serializable' 
WHEN 5 THEN 'Snapshot' END AS TRANSACTION_ISOLATION_LEVEL 
FROM sys.dm_exec_sessions 
where session_id = @@SPID
```
</details>

<details>
 <summary>Kill all connections but mine</summary>
 
```sql
DECLARE @kill varchar(8000) = '';

SELECT @kill = @kill + 'KILL ' + CONVERT(varchar(5), c.session_id) + ';'

FROM sys.dm_exec_connections AS c
JOIN sys.dm_exec_sessions AS s
    ON c.session_id = s.session_id
WHERE c.session_id <> @@SPID
--WHERE status = 'sleeping'
ORDER BY c.connect_time ASC

PRINT @kill

EXEC(@kill)
```
</details>

## Server specific queries

<details>
 <summary>SQL Server Version & Edition info</summary>
	
```sql
SELECT
CASE 
		WHEN CONVERT(VARCHAR(128), SERVERPROPERTY ('productversion')) like '8%' THEN 'SQL2000'
		WHEN CONVERT(VARCHAR(128), SERVERPROPERTY ('productversion')) like '9%' THEN 'SQL2005'
		WHEN CONVERT(VARCHAR(128), SERVERPROPERTY ('productversion')) like '10.0%' THEN 'SQL2008'
		WHEN CONVERT(VARCHAR(128), SERVERPROPERTY ('productversion')) like '10.5%' THEN 'SQL2008 R2'
		WHEN CONVERT(VARCHAR(128), SERVERPROPERTY ('productversion')) like '11%' THEN 'SQL2012'
		WHEN CONVERT(VARCHAR(128), SERVERPROPERTY ('productversion')) like '12%' THEN 'SQL2014'
		WHEN CONVERT(VARCHAR(128), SERVERPROPERTY ('productversion')) like '13%' THEN 'SQL2016'     
		WHEN CONVERT(VARCHAR(128), SERVERPROPERTY ('productversion')) like '14%' THEN 'SQL2017' 
		WHEN CONVERT(VARCHAR(128), SERVERPROPERTY ('productversion')) like '15%' THEN 'SQL2019' 
		ELSE 'unknown'
END AS MajorVersion,
SERVERPROPERTY('ProductLevel') AS ProductLevel,
SERVERPROPERTY('Edition') AS Edition,
SERVERPROPERTY('ProductVersion') AS ProductVersion
```
</details>

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

## Extended Events

<details>
 <summary>Any rpc_completed event greater than 1 second - parsed from ring buffer</summary>
 
 ```sql
SELECT targetdata = CAST(xet.target_data AS xml)
INTO #capture_waits_data
FROM sys.dm_xe_database_session_targets AS xet
JOIN sys.dm_xe_database_sessions AS xe
ON (xe.address = xet.event_session_address)
WHERE xe.name = 'QueryWaitTime_gt_1secs'
AND xet.target_name = 'ring_buffer'
SELECT xed.event_data.value('(@timestamp)[1]', 'datetime2') AS [timestamp],
xed.event_data.value('(data[@name="cpu_time"]/value)[1]', 'int') AS cpu_time_ms,
	floor(xed.event_data.value('(data[@name="cpu_time"]/value)[1]', 'int') / (1000000)) as seconds,
xed.event_data.value('(data[@name="logical_reads"]/value)[1]', 'int') AS logical_reads,
xed.event_data.value('(data[@name="row_count"]/value)[1]', 'int') AS row_count,
xed.event_data.value('(data[@name="duration"]/value)[1]', 'int') AS wait_type_duration_ms,
floor(xed.event_data.value('(data[@name="duration"]/value)[1]', 'int') / (1000000)) as seconds,
xed.event_data.value('(data[@name="result"]/text)[1]', 'varchar(max)') AS result,
xed.event_data.value('(data[@name="statement"]/value)[1]', 'varchar(max)') AS statement_text
FROM #capture_waits_data
CROSS APPLY targetdata.nodes('//RingBufferTarget/event') AS xed (event_data)
ORDER BY wait_type_duration_ms DESC
DROP TABLE #capture_waits_data
GO
```
</details>

<details>
 <summary>Create session - SP taking greater than 1 second</summary>
 
```sql
CREATE EVENT SESSION [<SPNAME>_gt_1secs] ON DATABASE 
ADD EVENT sqlserver.rpc_completed(
    ACTION(sqlserver.client_app_name,sqlserver.client_hostname,sqlserver.num_response_rows,sqlserver.sql_text,sqlserver.username)
    WHERE ([duration]>(1000000) AND [object_name] like '%<SPNAME>%'))
ADD TARGET package0.ring_buffer
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=OFF)
GO
```
</details>

## Azure SQL Single Database Only

<details>
 <summary>Tell me where it hurts (Azure SQL)</summary>
 
```sql
-- Isolate top waits for this database since last restart or failover (Query 24) (Top DB Waits)
WITH [Waits]
AS (SELECT wait_type, wait_time_ms/ 1000.0 AS [WaitS],
          (wait_time_ms - signal_wait_time_ms) / 1000.0 AS [ResourceS],
           signal_wait_time_ms / 1000.0 AS [SignalS],
           waiting_tasks_count AS [WaitCount],
           100.0 *  wait_time_ms / SUM (wait_time_ms) OVER() AS [Percentage],
           ROW_NUMBER() OVER(ORDER BY wait_time_ms DESC) AS [RowNum]
    FROM sys.dm_db_wait_stats WITH (NOLOCK)
    WHERE [wait_type] NOT IN (
        N'BROKER_EVENTHANDLER', N'BROKER_RECEIVE_WAITFOR', N'BROKER_TASK_STOP',
        N'BROKER_TO_FLUSH', N'BROKER_TRANSMITTER', N'CHECKPOINT_QUEUE',
        N'CHKPT', N'CLR_AUTO_EVENT', N'CLR_MANUAL_EVENT', N'CLR_SEMAPHORE',
        N'DBMIRROR_DBM_EVENT', N'DBMIRROR_EVENTS_QUEUE', N'DBMIRROR_WORKER_QUEUE',
        N'DBMIRRORING_CMD', N'DIRTY_PAGE_POLL', N'DISPATCHER_QUEUE_SEMAPHORE',
        N'EXECSYNC', N'FSAGENT', N'FT_IFTS_SCHEDULER_IDLE_WAIT', N'FT_IFTSHC_MUTEX',
        N'HADR_CLUSAPI_CALL', N'HADR_FILESTREAM_IOMGR_IOCOMPLETION', N'HADR_LOGCAPTURE_WAIT',
        N'HADR_NOTIFICATION_DEQUEUE', N'HADR_TIMER_TASK', N'HADR_WORK_QUEUE',
        N'KSOURCE_WAKEUP', N'LAZYWRITER_SLEEP', N'LOGMGR_QUEUE',
        N'MEMORY_ALLOCATION_EXT', N'ONDEMAND_TASK_QUEUE',
        N'PREEMPTIVE_HADR_LEASE_MECHANISM', N'PREEMPTIVE_SP_SERVER_DIAGNOSTICS',
        N'PREEMPTIVE_ODBCOPS',
        N'PREEMPTIVE_OS_LIBRARYOPS', N'PREEMPTIVE_OS_COMOPS', N'PREEMPTIVE_OS_CRYPTOPS',
        N'PREEMPTIVE_OS_PIPEOPS', N'PREEMPTIVE_OS_AUTHENTICATIONOPS',
        N'PREEMPTIVE_OS_GENERICOPS', N'PREEMPTIVE_OS_VERIFYTRUST',
        N'PREEMPTIVE_OS_FILEOPS', N'PREEMPTIVE_OS_DEVICEOPS', N'PREEMPTIVE_OS_QUERYREGISTRY',
        N'PREEMPTIVE_OS_WRITEFILE',
        N'PREEMPTIVE_XE_CALLBACKEXECUTE', N'PREEMPTIVE_XE_DISPATCHER',
        N'PREEMPTIVE_XE_GETTARGETSTATE', N'PREEMPTIVE_XE_SESSIONCOMMIT',
        N'PREEMPTIVE_XE_TARGETINIT', N'PREEMPTIVE_XE_TARGETFINALIZE',
        N'PREEMPTIVE_XHTTP',
        N'PWAIT_ALL_COMPONENTS_INITIALIZED', N'PWAIT_DIRECTLOGCONSUMER_GETNEXT',
        N'QDS_PERSIST_TASK_MAIN_LOOP_SLEEP',
        N'QDS_ASYNC_QUEUE',
        N'QDS_CLEANUP_STALE_QUERIES_TASK_MAIN_LOOP_SLEEP', N'REQUEST_FOR_DEADLOCK_SEARCH',
        N'RESOURCE_GOVERNOR_IDLE',
        N'RESOURCE_QUEUE', N'SERVER_IDLE_CHECK', N'SLEEP_BPOOL_FLUSH', N'SLEEP_DBSTARTUP',
        N'SLEEP_DCOMSTARTUP', N'SLEEP_MASTERDBREADY', N'SLEEP_MASTERMDREADY',
        N'SLEEP_MASTERUPGRADED', N'SLEEP_MSDBSTARTUP', N'SLEEP_SYSTEMTASK', N'SLEEP_TASK',
        N'SLEEP_TEMPDBSTARTUP', N'SNI_HTTP_ACCEPT', N'SP_SERVER_DIAGNOSTICS_SLEEP',
        N'SQLTRACE_BUFFER_FLUSH', N'SQLTRACE_INCREMENTAL_FLUSH_SLEEP', N'SQLTRACE_WAIT_ENTRIES',
        N'WAIT_FOR_RESULTS', N'WAITFOR', N'WAITFOR_TASKSHUTDOWN', N'WAIT_XTP_HOST_WAIT',
        N'WAIT_XTP_OFFLINE_CKPT_NEW_LOG', N'WAIT_XTP_CKPT_CLOSE', N'WAIT_XTP_RECOVERY',
        N'XE_BUFFERMGR_ALLPROCESSED_EVENT', N'XE_DISPATCHER_JOIN',
        N'XE_DISPATCHER_WAIT', N'XE_LIVE_TARGET_TVF', N'XE_TIMER_EVENT')
    AND waiting_tasks_count > 0)
SELECT
    MAX (W1.wait_type) AS [WaitType],
    CAST (MAX (W1.Percentage) AS DECIMAL (5,2)) AS [Wait Percentage],
    CAST ((MAX (W1.WaitS) / MAX (W1.WaitCount)) AS DECIMAL (16,4)) AS [AvgWait_Sec],
    CAST ((MAX (W1.ResourceS) / MAX (W1.WaitCount)) AS DECIMAL (16,4)) AS [AvgRes_Sec],
    CAST ((MAX (W1.SignalS) / MAX (W1.WaitCount)) AS DECIMAL (16,4)) AS [AvgSig_Sec],
    CAST (MAX (W1.WaitS) AS DECIMAL (16,2)) AS [Total_Wait_Sec],
    CAST (MAX (W1.ResourceS) AS DECIMAL (16,2)) AS [Resource_Sec],
    CAST (MAX (W1.SignalS) AS DECIMAL (16,2)) AS [Signal_Sec],
    MAX (W1.WaitCount) AS [Wait Count]  
FROM Waits AS W1
INNER JOIN Waits AS W2
ON W2.RowNum <= W1.RowNum
GROUP BY W1.RowNum
HAVING SUM (W2.Percentage) - MAX (W1.Percentage) < 99 -- percentage threshold
OPTION (RECOMPILE);
------
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

<details>
 <summary>Generate Index rename scripts for auto generated indexes (Azure SQL)</summary>
 
```sql
declare @SchemaName varchar(100)declare @TableName varchar(256)
declare @IndexName varchar(256)
declare @ColumnName varchar(100)
declare @is_unique varchar(100)
declare @IndexTypeDesc varchar(100)
declare @FileGroupName varchar(100)
declare @is_disabled varchar(100)
declare @IndexOptions varchar(max)
declare @IndexColumnId int
declare @IsDescendingKey int 
declare @IsIncludedColumn int
declare @TSQLScripCreationIndex varchar(max)
declare @TSQLScripDisableIndex varchar(max)
declare @TSQLScriptRenameIndex varchar(max)
declare @splitSQL varchar(max)

declare CursorIndex cursor for
 select schema_name(t.schema_id) [schema_name], t.name, ix.name,
 case when ix.is_unique = 1 then 'UNIQUE ' else '' END 
 , ix.type_desc,
 case when ix.is_padded=1 then 'PAD_INDEX = ON, ' else 'PAD_INDEX = OFF, ' end
 + case when ix.allow_page_locks=1 then 'ALLOW_PAGE_LOCKS = ON, ' else 'ALLOW_PAGE_LOCKS = OFF, ' end
 + case when ix.allow_row_locks=1 then  'ALLOW_ROW_LOCKS = ON, ' else 'ALLOW_ROW_LOCKS = OFF, ' end
 + case when INDEXPROPERTY(t.object_id, ix.name, 'IsStatistics') = 1 then 'STATISTICS_NORECOMPUTE = ON, ' else 'STATISTICS_NORECOMPUTE = OFF, ' end
 + case when ix.ignore_dup_key=1 then 'IGNORE_DUP_KEY = ON, ' else 'IGNORE_DUP_KEY = OFF, ' end
 + 'SORT_IN_TEMPDB = OFF, FILLFACTOR =' + CAST(ix.fill_factor AS VARCHAR(3)) AS IndexOptions
 , ix.is_disabled , FILEGROUP_NAME(ix.data_space_id) FileGroupName
 from sys.tables t 
 inner join sys.indexes ix on t.object_id=ix.object_id
 where ix.type>0 and ix.is_primary_key=0 and ix.is_unique_constraint=0 --and schema_name(tb.schema_id)= @SchemaName and tb.name=@TableName
 and t.is_ms_shipped=0 and t.name<>'sysdiagrams' and ix.name LIKE 'nci_wi%'
 order by schema_name(t.schema_id), t.name, ix.name

open CursorIndex
fetch next from CursorIndex into  @SchemaName, @TableName, @IndexName, @is_unique, @IndexTypeDesc, @IndexOptions,@is_disabled, @FileGroupName

while (@@fetch_status=0)
begin
 declare @IndexColumns varchar(max)
 declare @IncludedColumns varchar(max)
 
 set @IndexColumns=''
 set @IncludedColumns=''
 
 declare CursorIndexColumn cursor for 
  select col.name, ixc.is_descending_key, ixc.is_included_column
  from sys.tables tb 
  inner join sys.indexes ix on tb.object_id=ix.object_id
  inner join sys.index_columns ixc on ix.object_id=ixc.object_id and ix.index_id= ixc.index_id
  inner join sys.columns col on ixc.object_id =col.object_id  and ixc.column_id=col.column_id
  where ix.type>0 and (ix.is_primary_key=0 or ix.is_unique_constraint=0)
  and schema_name(tb.schema_id)=@SchemaName and tb.name=@TableName and ix.name=@IndexName
  order by ixc.index_column_id
 
 open CursorIndexColumn 
 fetch next from CursorIndexColumn into  @ColumnName, @IsDescendingKey, @IsIncludedColumn
 
 while (@@fetch_status=0)
 begin
  if @IsIncludedColumn=0 
   --set @IndexColumns=@IndexColumns + @ColumnName  + case when @IsDescendingKey=1  then ' DESC, ' else  ' ASC, ' end
   set @IndexColumns=@IndexColumns + @ColumnName  + '_'
  else 
	set @IncludedColumns=@IncludedColumns  + @ColumnName  +'_'
   --set @IncludedColumns=@IncludedColumns  + @ColumnName  +', ' 

  fetch next from CursorIndexColumn into @ColumnName, @IsDescendingKey, @IsIncludedColumn
 end

 close CursorIndexColumn
 deallocate CursorIndexColumn

 set @IndexColumns = substring(@IndexColumns, 1, len(@IndexColumns)-1)
 set @IncludedColumns = case when len(@IncludedColumns) >0 then substring(@IncludedColumns, 1, len(@IncludedColumns)-1) else '' end
  --print @IndexColumns
  --print @IncludedColumns

 set @TSQLScripCreationIndex =''
 set @TSQLScripDisableIndex =''
 set @TSQLScriptRenameIndex =''
 set @splitSQL =''

 set @TSQLScripCreationIndex='CREATE '+ @is_unique  +@IndexTypeDesc + ' INDEX ' +QUOTENAME(@IndexName)+' ON ' + QUOTENAME(@SchemaName) +'.'+ QUOTENAME(@TableName)+ '('+@IndexColumns+') '+ 
  --case when len(@IncludedColumns)>0 then CHAR(13) +'INCLUDE (' + @IncludedColumns+ ')' else '' end 
  case when len(@IncludedColumns)>0 then 'INCLUDE (' + @IncludedColumns+ ')' else '' end 
  --+ CHAR(13) + CHAR(10)+'WITH (' + @IndexOptions+ ') ON ' + QUOTENAME(@FileGroupName) 
  + ';'  


  --SET @splitSQL = (SELECT count(value) FROM STRING_SPLIT(@IncludedColumns, '_'))
  --SELECT @splitSQL

  IF (@IndexName <> 'IX_' + @IndexColumns 
  + case when len(@IncludedColumns)>0 then 
		case when (SELECT count(value) FROM STRING_SPLIT(@IncludedColumns, '_')) < 5 then 
			'_Inc_' + @IncludedColumns 
		else 
			'_Inc_Many'
		end
	else '' end)
BEGIN

  
  SET @TSQLScriptRenameIndex = 
	  'IF  EXISTS (SELECT * FROM sys.indexes WHERE name=''' + @IndexName + ''' AND object_id = OBJECT_ID(''' + @SchemaName + '.' + @TableName + ''')) ' + CHAR(13) + CHAR(10)
	  +'BEGIN' + CHAR(13) + CHAR(10)
	  +'EXECUTE sp_rename N''' + @SchemaName + '.' + @TableName + '.' + + @IndexName + ''', N''IX_' + @IndexColumns 
	  + case when len(@IncludedColumns)>0 then 
			case when (SELECT count(value) FROM STRING_SPLIT(@IncludedColumns, '_')) < 5 then
				'_Inc_' + @IncludedColumns 
			else 
				'_Inc_Many'
			end
		else '' end
	  + ''', N''INDEX''' 
	  + ';'  + CHAR(13) + CHAR(10) + 'END'
	  + CHAR(13) + CHAR(10) + 'GO'
	  + CHAR(13) + CHAR(10)
  END
  ELSE
BEGIN
	
  SET @TSQLScriptRenameIndex = '-- Skipped ' + @SchemaName + '.' + @TableName + '.' + + @IndexName
END


 if @is_disabled=1 
  set  @TSQLScripDisableIndex=  CHAR(13) +'ALTER INDEX ' +QUOTENAME(@IndexName) + ' ON ' + QUOTENAME(@SchemaName) +'.'+ QUOTENAME(@TableName) + ' DISABLE;' + CHAR(13) + CHAR(10) 

--print @TSQLScripCreationIndex
 print @TSQLScriptRenameIndex
 --print @TSQLScripDisableIndex

 fetch next from CursorIndex into  @SchemaName, @TableName, @IndexName, @is_unique, @IndexTypeDesc, @IndexOptions,@is_disabled, @FileGroupName

end
close CursorIndex
deallocate CursorIndex
```
</details>
