```SQL
CREATE OR ALTER PROCEDURE [dbo].[sp_smart_get_attendance_report]

(

@nId_User INT, -- Id de usuario consultor

@SNombreColaborador nvarchar(max), -- Filtro de colaborador

@sSlugToGetList nvarchar(max), -- Tipo de listado

@fechaIni VARCHAR(MAX), -- Filtro de fecha inicio

@fechaFin VARCHAR(MAX), -- Filtro de fecha fin

@sFilterOne nvarchar(max), -- Filtro por equipo

@sFilterTwo nvarchar(max), -- Filtro por estado de la asistencia

@sFilterThree NVARCHAR(max), -- Filtro por permisos

@sFilterFour nvarchar(max), -- Filtro por tipo modalidad

@sFilterNotification nvarchar(max), -- Filtro por Id de Asistencia

@campo VARCHAR(MAX), -- Nombre de columna de ordenamiento **Obligatorio**

@order VARCHAR(MAX) -- asc || desc **Obligatorio**

)

AS

BEGIN

DECLARE @script AS nvarchar(max);

DECLARE @viewClause AS nvarchar(max);

DECLARE @whereClause AS nvarchar(max);

DECLARE @orderClause AS nvarchar(max);

DECLARE @paginationClause AS nvarchar(max);

DECLARE @contador AS nvarchar(max);

DECLARE @conditions_array TABLE(condition nvarchar(max))

DECLARE @condition AS nvarchar(max);

DECLARE @view AS nvarchar(max) = 'select * from v_Listado_Asistencias_Reporte';

DECLARE @count INT;

DECLARE @total INT

DECLARE @ParmDefinition NVARCHAR(max);

  

-- LISTADO

IF @sSlugToGetList = 'ASSISTENCE-LIST-INDIVIDUAL'

BEGIN

SET @viewClause = N'select * from ('+ @view +' where nId_User ='+CONVERT(varchar,@nId_User)+')a';

END

  

ELSE IF @sSlugToGetList = 'TEAM-HAVE-SUBORDINATE'

BEGIN

DECLARE @CollaboratorsList VARCHAR(MAX);

DECLARE @nId_Coll INT = (SELECT c.nId_Colaborador FROM Colaboradores c

JOIN Usuarios u ON u.nId_Persona = c.nId_Persona

WHERE u.nId_Usuario = @nId_User)

SET @CollaboratorsList = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @nId_Coll),0,0)

  

SET @viewClause = N'select * from (select * from v_Listado_Asistencias_Reporte where nId_Colaborador IN ('+ @CollaboratorsList +'))a';

END

  

ELSE IF @sSlugToGetList = 'ASSISTENCE-LIST-PROYECTO'

BEGIN

SET @viewClause = N'select * from ('+ @view +' where nId_User_Lider ='+CONVERT(varchar,@nId_User)+' OR nId_User_Director = '+CONVERT(VARCHAR,@nId_User)+')a';

END

  

ELSE IF @sSlugToGetList = 'ASSISTENCE-LIST-ALL'

BEGIN

SET @viewClause = @view;

END

  

-- FILTROS

IF @SNombreColaborador IS NOT NULL

BEGIN

EXEC sp_get_like_condition 'SNombre_Colaborador', @SNombreColaborador, @condition OUTPUT

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @fechaIni IS NOT NULL AND @fechaFin IS NOT NULL

BEGIN

EXEC sp_get_dates_condition 'dFecha_Registro_Asistencia', @fechaIni ,@fechaFin, @condition OUTPUT

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterOne IS NOT NULL

BEGIN

DECLARE @CollaboratorsListTeam VARCHAR(MAX);

SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)

SET @condition = 'nId_Colaborador in('+@CollaboratorsListTeam+')'

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterTwo IS NOT NULL

BEGIN

SET @condition = 'nEstado in('+@sFilterTwo+')'

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterThree IS NOT NULL

BEGIN

SET @condition = 'bPermiso in('+@sFilterThree+')'

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterFour IS NOT NULL

BEGIN

SET @condition = 'nTipo_Modalidad in('+@sFilterFour+')'

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sFilterNotification IS NOT NULL

BEGIN

SET @condition = 'nId in('+@sFilterNotification+')'

INSERT INTO @conditions_array VALUES(@condition)

END

  

-- WHERE

SELECT @count = COUNT(*) FROM @conditions_array;

if @count > 0

BEGIN

set @whereClause = N'where ';

END

  

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

  

  

EXEC sp_get_order_clause @campo, @order , 'dFecha_Registro_Asistencia' , @orderClause OUTPUT

  

SET @contador = concat('select @totalOut = count(*) from (',@viewClause,' ',@whereClause,')a');

SET @ParmDefinition = N'@totalOut int OUTPUT';

EXEC sp_executesql @contador, @ParmDefinition, @totalOut = @total OUTPUT

  

SET @script = concat ('select *,',@total,' as nTotal from (',@viewClause,' ',@whereClause,')a ',@orderClause,' ',@paginationClause);

EXECUTE sp_executesql @script

END