```SQL
CREATE PROCEDURE [dbo].[sp_smart_get_proyectos_footer]

(

@IdProyecto int, ---Obtener By IdProyecto

@nId_Usuario int,

@nTipoFilter int, ---1 Pry , 2 Rq

@sNombre_Filter varchar(max),

@sFilterThree nvarchar(max), ----Id Requerimiento (notificacion)

@fechaCreaIni varchar(max),

@fechaCreaFin varchar(max),

@sFilterOne nvarchar(max), --Id Coordinador

@sFilterTwo nvarchar(max)--Estado del proyecto

)

as

begin

--DECLARE @PryEstimacion AS nvarchar(max);

--DECLARE @PryActivos AS nvarchar(max);

--DECLARE @PryCompleto AS nvarchar(max);

DECLARE @whereClause AS nvarchar(max);

  

DECLARE @nId_Rol int = (select nId_Rol from Roles_Usuarios ru where ru.nId_Usuario = @nId_Usuario);

DECLARE @sSlug1 int = (select count(o.sSlug) from Permisos_Opciones po

join Opciones o on o.nId_Opcion = po.nId_Opcion

where po.nId_Rol = @nId_Rol and o.sSlug = 'PROJECTS-LIST-ALL'

and po.nEstado = 1);

DECLARE @IdColaborador int = (

select c.nId_Colaborador from Usuarios u

join Personas p on u.nId_Persona = p.nId_Persona

join Colaboradores c on c.nId_Persona = p.nId_Persona

where u.nId_Usuario = @nId_Usuario

);

IF @IdProyecto is not null

BEGIN

--Obtener Detalle del proyecto por id

SET @whereClause = N'nId_Proyecto = ' + CONVERT(varchar, @IdProyecto);

END

ELSE IF @nTipoFilter = 2 AND @sNombre_Filter IS NOT NULL AND @sFilterThree IS NULL ---FiltroRequerimiento

BEGIN

IF @sSlug1 > 0

BEGIN

---TIENE PERMISO PARA VER TODO Y BUSCA POR REQUERIMIENTOS

DECLARE @TempIdPryByNameRQAll TABLE (nId_Proyecto int)

INSERT INTO @TempIdPryByNameRQAll

SELECT DISTINCT

p.nId_Proyecto

FROM Proyectos p

JOIN Requerimientos r

ON p.nId_Proyecto = r.nId_Proyecto

WHERE r.sNombre LIKE '%' + @sNombre_Filter + '%' COLLATE Latin1_general_CI_AI

DECLARE @tabla_tempAll varchar(MAX);

SELECT

* INTO #proyectos_all_nombre_req_all

FROM @TempIdPryByNameRQAll;

SET @tabla_tempAll = '#proyectos_all_nombre_req_all'

SET @whereClause = N'nId_Proyecto in (select * from ' + @tabla_tempAll +')' + ' and nPrincipal = ' +CONVERT(nvarchar, 1 );

END

ELSE

BEGIN

--BUSCANDO POR REQUERIMIENTOS Y SOLO MUETRA LOS PROYECTOS DEL USUARIO LOGUEADO

DECLARE @TempIdPryByNameRQ TABLE (nId_Proyecto int)

INSERT INTO @TempIdPryByNameRQ

SELECT DISTINCT

p.nId_Proyecto

FROM Proyectos p

JOIN Requerimientos r on p.nId_Proyecto = r.nId_Proyecto

JOIN Proyecto_Lider pl on pl.nId_Proyecto = p.nId_Proyecto

WHERE (p.nId_Director = @IdColaborador or pl.nId_Lider = @IdColaborador)

AND r.sNombre like '%' + @sNombre_Filter + '%' COLLATE Latin1_general_CI_AI

  

DECLARE @tabla_temp varchar(MAX);

SELECT

* INTO #proyectos_all_nombre_req

FROM @TempIdPryByNameRQ;

SET @tabla_temp = '#proyectos_all_nombre_req'

SET @whereClause = N'nId_Proyecto in (select * from ' + @tabla_temp +')' + ' and nPrincipal = ' +CONVERT(nvarchar, 1 );

END

end

ELSE IF @sFilterThree IS NOT NULL

BEGIN

---OBTENEMOS EL ID DEL PROYECTO MEDIANTE EL ID DEL REQUERIMIENTO (NOTIFICACION)

DECLARE @TempIdPryByIdRq TABLE (nId_Proyecto int)

INSERT INTO @TempIdPryByIdRq

SELECT p.nId_Proyecto FROM

Proyectos p

JOIN Requerimientos r ON r.nId_Proyecto = p.nId_Proyecto

WHERE r.nId_Requerimiento IN (@sFilterThree)

BEGIN

DECLARE @tabla_temp_2 varchar(MAX);

SELECT

* INTO #proyectos_all_IdRq

FROM @TempIdPryByIdRq;

SET @tabla_temp_2 = '#proyectos_all_IdRq'

END

SET @whereClause = N'nId_Proyecto in (select * from ' + @tabla_temp_2 +')' + ' and nPrincipal = ' +CONVERT(nvarchar, 1 );

END

ELSE

BEGIN

IF @sSlug1 > 0

BEGIN

---CUENTA CON LA VISTA TODO LOS PROYECTOS

SET @whereClause = N'nPrincipal = ' +CONVERT(nvarchar, 1 );

END

ELSE

BEGIN

----MUESTRA LOS PROYECTOS SUYOS

SET @whereClause = N'nLider = ' + CONVERT(varchar, @IdColaborador)+ ' or nDirector = ' + CONVERT(varchar, @IdColaborador) + ' and nPrincipal = ' +CONVERT(nvarchar, 1 );

END

END

DECLARE @conditions AS TABLE (condition nvarchar(max));

DECLARE @conditionOUT AS nvarchar(max);

IF @fechaCreaIni IS NOT NULL AND @fechaCreaFin IS NOT NULL

BEGIN

EXEC sp_get_dates_condition 'dFecha_Creacion', @fechaCreaIni ,@fechaCreaFin, @conditionOUT OUTPUT

INSERT INTO @conditions VALUES(@conditionOUT)

END

IF @sNombre_Filter IS NOT NULL AND @nTipoFilter =1 AND @sFilterThree IS NULL

BEGIN

EXEC sp_get_like_condition 'sNombre_Proyecto_Lista', @sNombre_Filter, @conditionOUT OUTPUT

INSERT INTO @conditions VALUES(@conditionOUT)

END

IF @sFilterOne IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES( N'nCoordinador in('+@sFilterOne+')');

END

IF @sFilterTwo IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES('nEstado in('+@sFilterTwo+')');

END

DECLARE @conditionsClause AS nvarchar(max);

  

-- Concatenar las condiciones en una variable

SET @conditionsClause = (SELECT STRING_AGG(condition, ' AND ') FROM @conditions);

DECLARE @PryEstimacion AS INT;

DECLARE @PryActivos AS INT;

DECLARE @PryCompleto AS INT;

DECLARE @sqlQuery AS NVARCHAR(MAX);

  

IF @conditionsClause IS NULL

BEGIN

SET @sqlQuery =

'SELECT

@PryEstimacion = ISNULL(SUM(ISNULL(nEstimacion, 0)), 0),

@PryActivos = COUNT(CASE WHEN nEstado = 1 THEN 1 END),

@PryCompleto = COUNT(CASE WHEN nEstado = 3 THEN 1 END)

FROM

v_Listado_Proyectos_con_requerimientos

WHERE ' + @whereClause;

-- Execute the dynamic query and store the results in variables

EXEC sp_executesql @sqlQuery, N'@PryEstimacion INT OUTPUT, @PryActivos INT OUTPUT, @PryCompleto INT OUTPUT',

@PryEstimacion = @PryEstimacion OUTPUT,

@PryActivos = @PryActivos OUTPUT,

@PryCompleto = @PryCompleto OUTPUT;

END

ELSE

BEGIN

  

SET @sqlQuery =

'SELECT

@PryEstimacion = ISNULL(SUM(ISNULL(nEstimacion, 0)), 0),

@PryActivos = COUNT(CASE WHEN nEstado = 1 THEN 1 END),

@PryCompleto = COUNT(CASE WHEN nEstado = 3 THEN 1 END)

FROM

v_Listado_Proyectos_con_requerimientos

WHERE ' + @conditionsClause + ' AND ' + @whereClause;

-- Execute the dynamic query and store the results in variables

EXEC sp_executesql @sqlQuery, N'@PryEstimacion INT OUTPUT, @PryActivos INT OUTPUT, @PryCompleto INT OUTPUT',

@PryEstimacion = @PryEstimacion OUTPUT,

@PryActivos = @PryActivos OUTPUT,

@PryCompleto = @PryCompleto OUTPUT;

  

END

SELECT @PryEstimacion AS PryEstimacion,@PryActivos AS PryActivos ,@PryCompleto AS PryCompletos

END
```