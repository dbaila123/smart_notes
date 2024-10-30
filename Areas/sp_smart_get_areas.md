```sql
CREATE OR ALTER PROCEDURE [dbo].[sp_smart_get_areas]

(

@sNombreArea NVARCHAR(MAX) = NULL,

@fechaRegistro VARCHAR(MAX) = NULL,

@sFilterState NVARCHAR(MAX) = NULL,

@pnum INT,

@size INT,

@campo VARCHAR(MAX),

@order VARCHAR(MAX)

)

AS

BEGIN

SET NOCOUNT ON;

DECLARE @script NVARCHAR(MAX);

DECLARE @viewClause NVARCHAR(MAX);

DECLARE @whereClause NVARCHAR(MAX) = '';

DECLARE @orderClause NVARCHAR(MAX);

DECLARE @paginationClause NVARCHAR(MAX);

DECLARE @contador NVARCHAR(MAX);

DECLARE @total INT;

DECLARE @defaultOrderField VARCHAR(100) = 'nId_Area';

  

SET @viewClause = N'SELECT * FROM v_Listado_Areas';

  

DECLARE @conditions_array TABLE(condition NVARCHAR(MAX));

  

IF @sNombreArea IS NOT NULL

BEGIN

DECLARE @conditionNombre NVARCHAR(MAX);

EXEC sp_get_like_condition 'sNombre_Area', @sNombreArea, @conditionNombre OUTPUT;

INSERT INTO @conditions_array VALUES(@conditionNombre);

END

  

IF @fechaRegistro IS NOT NULL

BEGIN

DECLARE @conditionFecha NVARCHAR(MAX);

EXEC sp_get_dates_condition 'dDatetime_Creador', @fechaRegistro, @fechaRegistro, @conditionFecha OUTPUT;

INSERT INTO @conditions_array VALUES(@conditionFecha);

END

  

IF @sFilterState IS NOT NULL

BEGIN

INSERT INTO @conditions_array

VALUES(CONCAT('nEstado IN (', @sFilterState, ')'));

END

  

SELECT @whereClause =

CASE

WHEN COUNT(*) > 0

THEN 'WHERE ' + STRING_AGG(condition, ' AND ')

ELSE ''

END

FROM @conditions_array;

  

EXEC sp_get_order_clause @campo, @order, @defaultOrderField, @orderClause OUTPUT;

EXEC sp_get_pagination_clause @pnum, @size, @paginationClause OUTPUT;

  

SET @contador = CONCAT('SELECT @totalOut = COUNT(*) FROM (', @viewClause, ' ', @whereClause, ') AS a');

DECLARE @ParmDefinition NVARCHAR(MAX) = N'@totalOut INT OUTPUT';

EXEC sp_executesql @contador, @ParmDefinition, @totalOut = @total OUTPUT;

  

SET @script = CONCAT(

'SELECT *, ',

@total,

' AS nTotal FROM (',

@viewClause,

' ',

@whereClause,

') AS a ',

@orderClause,

' ',

@paginationClause

);

  

PRINT @script;

  

EXEC sp_executesql @script;

END;
```