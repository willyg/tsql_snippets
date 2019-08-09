Find indexes to rebuild/reorganize
```
SELECT
  OBJECT_SCHEMA_NAME(ps.object_id) as schema_name
  , OBJECT_NAME(ps.object_id) as table_name
  , i.name as index_name
  , ps.index_type_desc as index_type
  , ps.avg_fragmentation_in_percent
  , 'DECLARE @t1 DATETIME, @t2 INT; SET @t1 = GETDATE();' +
    'ALTER INDEX ' + QUOTENAME(i.name)  + ' ON ' + QUOTENAME(OBJECT_SCHEMA_NAME(ps.object_id)) + '.' + QUOTENAME(OBJECT_NAME(i.object_id)) +
    CASE
      WHEN ps.avg_fragmentation_in_percent > 30 THEN ' REBUILD;' 
      WHEN ps.avg_fragmentation_in_percent >= 5 THEN ' REORGANIZE;'
    END +
    'SET @t2 = DATEDIFF(MINUTE, @t1, GETDATE());' +
    'RAISERROR (''' + i.name + ' took %d minutes'', 10,1, @t2) WITH NOWAIT; ' as SQLQuery
FROM
  sys.dm_db_index_physical_stats(DB_ID(), null, null, null, null) ps
  JOIN sys.indexes i on ps.object_id = i.object_id and ps.index_id = i.index_id
WHERE
  i.index_id > 0
  and ps.page_count > 100
  and ps.avg_fragmentation_in_percent >= 5
ORDER BY
  ps.avg_fragmentation_in_percent DESC
```
