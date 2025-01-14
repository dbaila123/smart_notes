```sql
ALTER PROCEDURE [dbo].[sp_smart_get_collaborators_report]

(

@nId INT, -- Id Colaborador

@SNombreColaborador nvarchar(max),

@sSlugToGetList nvarchar(max),

----------------------------------------------------

@dFecha_Inicio VARCHAR(MAX),

@dFecha_Fin VARCHAR(MAX),

@sFilterOne nvarchar(max), --Filtro por modalidad

@sFilterTwo nvarchar(max), --Filtro por estado de colaborador

@sFilterThree NVARCHAR(max), --Filtro por equipos

@sFilterFour NVARCHAR(max), --Filtro por sedes

@sFilterFive NVARCHAR(max), --Filtro por cargos

@sFilterSix NVARCHAR(max), --Filtro por empresa

  

@campo VARCHAR(MAX), --obligatorio column name toOrder

@order VARCHAR(MAX) --obligatorio asc || desc

)

AS

BEGIN

-- david

set @sFilterFour = replace(@sFilterFour, '-', ',')

set @sFilterFive = replace(@sFilterFive, '-', ',')

set @sFilterSix = replace(@sFilterSix, '-', ',')

  

DECLARE @script AS nvarchar(max);

DECLARE @viewClause AS nvarchar(max);

DECLARE @whereClause AS nvarchar(max);

DECLARE @orderClause AS nvarchar(max);

DECLARE @paginationClause AS nvarchar(max);

DECLARE @contador AS nvarchar(max);

  

IF @sSlugToGetList = 'COLABORADOR-LISTA-INDIVIDUAL'

BEGIN

SET @viewClause = N'select * from (select * from v_listado_Colaborador_report where nId ='+CONVERT(varchar,@nId)+')a';

END

  

ELSE IF @sSlugToGetList = 'TEAM-HAVE-SUBORDINATE'

BEGIN

DECLARE @CollaboratorsList VARCHAR(MAX);

SET @CollaboratorsList = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @nId),0,0)

  

SET @viewClause = N'select * from (select * from v_listado_Colaborador_report where nId IN ( '+ @CollaboratorsList +' )) a';

END

  

ELSE IF @sSlugToGetList = 'COLABORADOR-LISTADO'

BEGIN

SET @viewClause = N'select * from v_listado_Colaborador_report';

END

  

--Set @whereClause

DECLARE @conditions_array TABLE(condition nvarchar(max))

DECLARE @condition AS nvarchar(max);

  

IF @SNombreColaborador IS NOT NULL

  

BEGIN

EXEC sp_get_like_condition 'sNombre_Colaborador', @SNombreColaborador, @condition OUTPUT

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @dFecha_Inicio IS NOT NULL AND @dFecha_Fin IS NOT NULL

BEGIN

EXEC sp_get_dates_condition 'dFec_Ingreso', @dFecha_Inicio ,@dFecha_Fin, @condition OUTPUT

INSERT INTO @conditions_array VALUES(@condition)

END

  

--SET @sFilterOne = REPLACE(@sFilterOne, '-', ',')

--SET @sFilterTwo = REPLACE(@sFilterTwo, '-', ',')

--SET @sFilterThree = REPLACE(@sFilterThree, '-', ',')

  

IF @sFilterOne IS NOT NULL

BEGIN

SET @condition = 'nTipo_Modalidad in('+@sFilterOne+')'

PRINT 'tipo modalidad'

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterTwo IS NOT NULL

BEGIN

SET @condition = 'nEstado in('+@sFilterTwo+')'

PRINT 'tipo estado'

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterThree IS NOT NULL

BEGIN

DECLARE @CollaboratorsListTeam VARCHAR(MAX);

SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterThree),0,0)

SET @condition = 'nId in('+@CollaboratorsListTeam+')'

PRINT 'equipo'

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterFour IS NOT NULL BEGIN

SET @condition = 'nId_Sede IN( '+@sFilterFour+')';

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterFive IS NOT NULL

BEGIN

SET @condition = 'nId_Cargo IN( '+@sFilterFive+')';

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterSix IS NOT NULL

BEGIN

SET @condition = 'nId_Empresa IN( '+@sFilterSix+')';

INSERT INTO @conditions_array VALUES(@condition)

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

  

  

PRINT 'whereClause end'

--Set @orderClause

EXEC sp_get_order_clause @campo, @order , 'dFec_Ingreso' , @orderClause OUTPUT

  

  

DECLARE @total INT=100

DECLARE @ParmDefinition NVARCHAR(max);

  

set @contador = concat('select @totalOut = count(*) from (',@viewClause,' ',@whereClause,')a');

SET @ParmDefinition = N'@totalOut int OUTPUT';

PRINT @contador

  

  

EXEC sp_executesql @contador, @ParmDefinition, @totalOut = @total OUTPUT

  

SET @script = CONCAT(

'select *, ', @total, ' as nTotal from (', @viewClause, ' ', @whereClause, ') a ', @orderClause, ' ', @paginationClause

);

  

  

insert into Log values(@script)

print 'hola'

print @script

print 'adios'

  

EXECUTE sp_executesql @script

END
```

