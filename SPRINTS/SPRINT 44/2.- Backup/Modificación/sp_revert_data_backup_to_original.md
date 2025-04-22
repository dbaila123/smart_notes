```sql 
CREATE PROCEDURE sp_revert_data_backup_to_original

(

@listable NVARCHAR(MAX)

)

AS

BEGIN

DECLARE @tableName NVARCHAR(MAX)

DECLARE @columns NVARCHAR(MAX)

DECLARE @sql NVARCHAR(MAX)

DECLARE @errorTables TABLE (TableName NVARCHAR(MAX), ErrorMessage NVARCHAR(MAX))

SELECT TRIM(value) as nombre

INTO #tablas

FROM STRING_SPLIT(@listable, ',')

  

EXEC sp_msforeachtable 'ALTER TABLE ? NOCHECK CONSTRAINT ALL';

  

DECLARE table_cursor CURSOR FOR

SELECT name

FROM sys.tables

WHERE type = 'U'

AND name NOT LIKE '%\_backup%' ESCAPE '\'

AND name NOT IN (SELECT nombre FROM #tablas)

OPEN table_cursor;

FETCH NEXT FROM table_cursor INTO @tableName;

  

WHILE @@FETCH_STATUS = 0

BEGIN

BEGIN TRY

SELECT @columns = STRING_AGG(QUOTENAME(name), ',')

FROM sys.columns

WHERE object_id = OBJECT_ID('dbo.' + @tableName);

  

SET @sql = '

DELETE FROM dbo.' + @tableName + ';

  

SET IDENTITY_INSERT dbo.' + @tableName + ' ON;

  

INSERT INTO dbo.' + @tableName + ' (' + @columns + ')

SELECT ' + @columns + ' FROM dbo.' + @tableName + '_backup;

  

SET IDENTITY_INSERT dbo.' + @tableName + ' OFF;

  

DROP TABLE dbo.' + @tableName + '_backup;';

  

EXEC sp_executesql @sql;

  

END TRY

BEGIN CATCH

INSERT INTO @errorTables (TableName, ErrorMessage)

VALUES (@tableName, ERROR_MESSAGE());

END CATCH

  

FETCH NEXT FROM table_cursor INTO @tableName;

END;

  

CLOSE table_cursor;

DEALLOCATE table_cursor;

  

EXEC sp_msforeachtable 'ALTER TABLE ? CHECK CONSTRAINT ALL';

  

SELECT * FROM @errorTables;

END;
```