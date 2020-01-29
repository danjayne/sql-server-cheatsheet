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
	  +'EXEC sp_rename N''' + @SchemaName + '.' + @TableName + '.' + + @IndexName + ''', N''IX_' + @IndexColumns 
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
 <summary>Extended Events Slow rpc_completed - read from ring buffer</summary>
 
 ```sql
  SELECT targetdata = CAST(xet.target_data AS xml)
  INTO #capture_waits_data
FROM sys.dm_xe_database_session_targets AS xet
JOIN sys.dm_xe_database_sessions AS xe
ON (xe.address = xet.event_session_address)
WHERE xe.name = 'QueryWaitTime_gt_1Second'
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
