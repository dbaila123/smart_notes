```SQL
CREATE PROCEDURE Sp_smart_getClosingDate

(

@nCantidadDias INT,

----------------------------------------------------

  

@pnum INT, --obligatorio --Número de Página

@size INT, --obligatorio --Cantidad de Registros por página

@campo VARCHAR(MAX), --obligatorio column name toOrder

@order VARCHAR(MAX) --obligatorio asc || desc

)

AS

BEGIN

DECLARE @script AS nvarchar(max);

DECLARE @viewClause AS nvarchar(max);

DECLARE @whereClause AS nvarchar(max);

DECLARE @orderClause AS nvarchar(max);

DECLARE @paginationClause AS nvarchar(max);

DECLARE @contador AS nvarchar(max);

  

SET @viewClause = N'SELECT * FROM v_Listado_ClosingDate';

  

-- --Set @whereClause

DECLARE @conditions_array TABLE(condition nvarchar(max))

DECLARE @condition AS nvarchar(max);

  

IF @nCantidadDias IS NOT NULL

BEGIN

-- Construye la condición dinámica para comparar la columna 'Dias' con el valor de @nCantidadDias

SET @condition = N'nCantidadDias <= ' + CAST(@nCantidadDias AS nvarchar(max));

-- Inserta la condición en la tabla de condiciones

INSERT INTO @conditions_array VALUES(@condition);

END

  

DECLARE @count INT;

  

SELECT @count = COUNT(*) FROM @conditions_array;

if @count > 0

begin

set @whereClause = N'where ';

end

  

WHILE @count > 0

BEGIN

DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions_array);

IF @count = 1

BEGIN

SET @whereClause = concat (@whereClause,@condition_temp);

END

ELSE

BEGIN

SET @whereClause = concat (@whereClause,@condition_temp,' and ');

END

DELETE TOP (1) FROM @conditions_array

SELECT @count = COUNT(*) FROM @conditions_array;

END

--END Set @whereClause

  

--Set @orderClause

EXEC sp_get_order_clause @campo, @order , 'nIdClosingDate' , @orderClause OUTPUT

--END Set @orderClause

  

--Set @paginationClause

EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT

--END Set @paginationClause

  

--set @contador-registros

DECLARE @total INT

DECLARE @ParmDefinition NVARCHAR(max);

  

set @contador = concat('select @totalOut = count(*) from (',@viewClause,' ',@whereClause,')a');

SET @ParmDefinition = N'@totalOut int OUTPUT';

  

EXEC sp_executesql @contador, @ParmDefinition, @totalOut = @total OUTPUT

-- --end @contador

  

-- --Set @script

SET @script = concat ('select *,',@total,' as nTotal from (',@viewClause,' ',@whereClause,')a ',@orderClause,' ',@paginationClause);

-- --END Set @script

  

print @script

EXECUTE sp_executesql @script

END
```