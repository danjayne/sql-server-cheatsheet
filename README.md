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

<details>
 <summary></summary>
 
 ```sql
	
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
