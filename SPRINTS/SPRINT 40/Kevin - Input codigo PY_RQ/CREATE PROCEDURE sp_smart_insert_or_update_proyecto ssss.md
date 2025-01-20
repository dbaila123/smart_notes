```SQL
ALTER OR ALTER PROCEDURE [dbo].[sp_smart_insert_or_update_proyecto]

(

@IdProyecto int, ---Obtener By IdProyecto

@nId_Usuario int,

@nTipoFilter int, ---1 Pry , 2 Rq

@sNombre_Filter varchar(max),

@sFilterThree nvarchar(max), ----Id Requerimiento (notificacion)

@sFilterFour nvarchar(max),

@fechaCreaIni varchar(max),

@fechaCreaFin varchar(max),

@sFilterOne nvarchar(max), --Id Coordinador

@sFilterTwo nvarchar(max), --Estado del proyecto

@sFilterFive varchar(max), --scodigo-----------------------------------------

@pnum int, --obligatorio --Número de Páginas

@size int, --obligatorio --Cantidad de Registros por página

@campo varchar(MAX), --obligatorio column name toOrder

@order varchar(MAX) --obligatorio asc || desc

)

as

begin

DECLARE @script AS nvarchar(max)

DECLARE @whereClause AS nvarchar(max)

DECLARE @orderClause AS nvarchar(max)

DECLARE @paginationClause AS nvarchar(max)

DECLARE @selectClause AS nvarchar(max)

IF @IdProyecto is not null

BEGIN

SET @selectClause = 'SELECT V.*,

V.sNombre_Proyecto_Detalle as sNombre_Proyecto,

(SELECT COUNT(*) FROM v_Listado_Proyectos_con_requerimientos WHERE '

END

ELSE

BEGIN

SET @selectClause = 'SELECT V.*,

V.sNombre_Proyecto_Lista as sNombre_Proyecto,

(SELECT COUNT(*) FROM v_Listado_Proyectos_con_requerimientos WHERE '

END

  

DECLARE @nId_Rol int = (select nId_Rol from Roles_Usuarios ru where ru.nId_Usuario = @nId_Usuario)

  

DECLARE @sSlug1 int = (select count(o.sSlug) from Permisos_Opciones po

  

join Opciones o on o.nId_Opcion = po.nId_Opcion

where po.nId_Rol = @nId_Rol and o.sSlug = 'PROJECTS-LIST-ALL'

and po.nEstado = 1)

  

DECLARE @IdColaborador int = (

select c.nId_Colaborador from Usuarios u

join Personas p on u.nId_Persona = p.nId_Persona

join Colaboradores c on c.nId_Persona = p.nId_Persona

where u.nId_Usuario = @nId_Usuario

)

  

IF @IdProyecto is not null

BEGIN

--Obtener Detalle del proyecto por id

SET @whereClause = N'nId_Proyecto = ' + CONVERT(varchar, @IdProyecto)

END

ELSE IF @nTipoFilter = 2 AND @sNombre_Filter IS NOT NULL AND @sFilterThree IS NULL AND @sFilterFour IS NULL ---FiltroRequerimiento

BEGIN

IF @sSlug1 > 0

BEGIN

---TIENE PERMISO PARA VER TODO Y BUSCA POR REQUERIMIENTOS

print 'entro1'

DECLARE @TempIdPryByNameRQAll TABLE (nId_Proyecto int)

INSERT INTO @TempIdPryByNameRQAll

SELECT DISTINCT

p.nId_Proyecto

FROM Proyectos p

JOIN v_Listado_Requerimientos_refact r

ON p.nId_Proyecto = r.nId_Proyecto

WHERE

r.sNombre_Requerimiento LIKE '%' + @sNombre_Filter + '%' COLLATE Latin1_general_CI_AI

DECLARE @tabla_tempAll varchar(MAX)

SELECT

* INTO #proyectos_all_nombre_req_all

FROM @TempIdPryByNameRQAll

SET @tabla_tempAll = '#proyectos_all_nombre_req_all'

  

SET @whereClause = N'nId_Proyecto in (select * from ' + @tabla_tempAll +')' + ' and nPrincipal = ' +CONVERT(nvarchar, 1 )

END

ELSE

BEGIN

--BUSCANDO POR REQUERIMIENTOS Y SOLO MUETRA LOS PROYECTOS DEL USUARIO LOGUEADO

DECLARE @TempIdPryByNameRQ TABLE (nId_Proyecto int)

INSERT INTO @TempIdPryByNameRQ

SELECT DISTINCT

p.nId_Proyecto

FROM Proyectos p

JOIN v_Listado_Requerimientos_refact r on p.nId_Proyecto = r.nId_Proyecto

JOIN Proyecto_Lider pl on pl.nId_Proyecto = p.nId_Proyecto

WHERE (p.nId_Director = @IdColaborador or pl.nId_Lider = @IdColaborador)

AND r.sNombre_Requerimiento like '%' + @sNombre_Filter + '%' COLLATE Latin1_general_CI_AI

  

DECLARE @tabla_temp varchar(MAX)

SELECT

* INTO #proyectos_all_nombre_req

FROM @TempIdPryByNameRQ

SET @tabla_temp = '#proyectos_all_nombre_req'

  

SET @whereClause = N'nId_Proyecto in (select * from ' + @tabla_temp +')' + ' and nPrincipal = ' +CONVERT(nvarchar, 1 )

END

END

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

DECLARE @tabla_temp_2 varchar(MAX)

  

SELECT

* INTO #proyectos_all_IdRq

FROM @TempIdPryByIdRq

SET @tabla_temp_2 = '#proyectos_all_IdRq'

END

SET @whereClause = N'nId_Proyecto in (select * from ' + @tabla_temp_2 +')' + ' and nPrincipal = ' +CONVERT(nvarchar, 1 )

END

ELSE IF @sFilterFour IS NOT NULL

BEGIN

---OBTENEMOS EL ID DEL PROYECTO MEDIANTE EL ID DEL PROYECTO (NOTIFICACION)

DECLARE @TempIdPryByIdPry TABLE (nId_Proyecto int) --@TempIdPryByIdRq

INSERT INTO @TempIdPryByIdPry

SELECT p.nId_Proyecto

FROM

Proyectos p

WHERE p.nId_Proyecto IN (@sFilterFour)

  

BEGIN

DECLARE @tabla_temp_3 varchar(MAX)

SELECT

* INTO #proyectos_all_IdPry

FROM @TempIdPryByIdPry

SET @tabla_temp_3 = '#proyectos_all_IdPry'

END

SET @whereClause = N'nId_Proyecto in (select * from ' + @tabla_temp_3 +')' + ' and nPrincipal = ' +CONVERT(nvarchar, 1 )

END

ELSE

BEGIN

IF @sSlug1 > 0

BEGIN

---CUENTA CON LA VISTA TODO LOS PROYECTOS

SET @whereClause = N'nPrincipal = ' +CONVERT(nvarchar, 1 )

END

ELSE

BEGIN

----MUESTRA LOS PROYECTOS SUYOS

SET @whereClause = N'nLider = ' + CONVERT(varchar, @IdColaborador)+ ' or nDirector = ' + CONVERT(varchar, @IdColaborador) + ' and nPrincipal = ' +CONVERT(nvarchar, 1 )

END

END

DECLARE

@conditions AS TABLE (condition nvarchar(max))

DECLARE @conditionOUT AS nvarchar(max)

  

IF @fechaCreaIni IS NOT NULL AND @fechaCreaFin IS NOT NULL

BEGIN

EXEC sp_get_dates_condition 'dFecha_Creacion', @fechaCreaIni ,@fechaCreaFin, @conditionOUT OUTPUT

  

INSERT INTO @conditions VALUES(@conditionOUT)

END

  

-------------------------------------------------------------------------------------------------------------------

IF @sFilterFive IS NOT NULL

BEGIN

-- Puedes usar sp_get_like_condition para búsquedas parciales

EXEC sp_get_like_condition 'sCodigo', @sFilterFive, @conditionOUT OUTPUT

INSERT INTO @conditions VALUES(@conditionOUT)

  

-- O si prefieres una búsqueda exacta, puedes usar:

-- INSERT INTO @conditions VALUES('sCodigo = ''' + @sFilterFive + '''')

END

  

-------------------------------------------------------------------------------------------------------------------

  

  

IF @sNombre_Filter IS NOT NULL AND @nTipoFilter =1 AND @sFilterThree IS NULL

BEGIN

EXEC sp_get_like_condition 'sNombre_Proyecto_Lista', @sNombre_Filter, @conditionOUT OUTPUT

INSERT INTO @conditions VALUES(@conditionOUT)

END

  

IF @sFilterOne IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES( N'nCoordinador in('+@sFilterOne+')')

END

  

IF @sFilterTwo IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES('nEstado in('+@sFilterTwo+')')

END

  

-- Construir la cláusula ORDER BY

SET @orderClause = 'ORDER BY ' + @campo + ' ' + @order

  

-- Construir la cláusula OFFSET FETCH

SET @paginationClause = 'OFFSET ' + CAST((@pnum - 1) * @size AS VARCHAR) + ' ROWS FETCH NEXT ' + CAST(@size AS VARCHAR) + ' ROWS ONLY'

  

DECLARE @conditionsClause AS nvarchar(max)

  

-- Concatenar las condiciones en una variable

SET @conditionsClause = (SELECT STRING_AGG(condition, ' AND ') FROM @conditions)

  

IF @conditionsClause IS NULL --obtener por Id

BEGIN

SET @script = @selectClause + @whereClause + ') AS total

,(SELECT COUNT(*) from Proyecto_Lider

WHERE nId_Proyecto = V.nId_Proyecto

AND nEstado = 1) AS nlideres

,(SELECT ISNULL(STRING_AGG(P.sPrimer_Nombre +'' '' +P.sApe_Paterno,'',''),'''') LIDERES

FROM Proyecto_Lider PL

INNER JOIN Colaboradores C ON PL.nId_Lider = C.nId_Colaborador

LEFT JOIN Personas P ON P.nId_Persona = C.nId_Persona

WHERE PL.nId_Proyecto = V.NID_PROYECTO AND PL.nEstado = 1

AND PL.nPrincipal = 0) AS slideres

,(SELECT STRING_AGG(VLP.nlider,'','') FROM v_Listado_Proyectos_con_requerimientos VLP WHERE VLP.nId_Proyecto = V.nId_Proyecto) AS snidlideres

FROM v_Listado_Proyectos_con_requerimientos V WHERE ' + @whereClause

END

ELSE IF @pnum IS NULL AND @size IS NULL ----Descarga

BEGIN

SET @script = @selectClause + @conditionsClause + ' AND ' + @whereClause + ') AS total

,(SELECT COUNT(*) from Proyecto_Lider

WHERE nId_Proyecto = V.nId_Proyecto

AND nEstado = 1) AS nlideres

,(SELECT ISNULL(STRING_AGG(P.sPrimer_Nombre +'' '' +P.sApe_Paterno,'',''),'''') LIDERES

FROM Proyecto_Lider PL

INNER JOIN Colaboradores C ON PL.nId_Lider = C.nId_Colaborador

LEFT JOIN Personas P ON P.nId_Persona = C.nId_Persona

WHERE PL.nId_Proyecto = V.NID_PROYECTO AND PL.nEstado = 1

AND PL.nPrincipal = 0) AS slideres

,(SELECT

STRING_AGG(VLP.nlider,'','') FROM v_Listado_Proyectos_con_requerimientos VLP WHERE VLP.nId_Proyecto = V.nId_Proyecto) AS snidlideres

FROM v_Listado_Proyectos_con_requerimientos V WHERE ' + @conditionsClause + ' AND ' + @whereClause + ' ' + @orderClause

END

ELSE

BEGIN

SET @script = @selectClause + @conditionsClause + ' AND ' + @whereClause + ') AS total

,(SELECT COUNT(*) from Proyecto_Lider

WHERE nId_Proyecto = V.nId_Proyecto

AND nEstado = 1) AS nlideres ,(SELECT ISNULL(STRING_AGG(P.sPrimer_Nombre +'' '' +P.sApe_Paterno,'',''),'''') LIDERES

FROM Proyecto_Lider PL

INNER JOIN Colaboradores C ON PL.nId_Lider = C.nId_Colaborador

LEFT JOIN Personas P ON P.nId_Persona = C.nId_Persona

  

WHERE PL.nId_Proyecto = V.NID_PROYECTO AND PL.nEstado = 1

AND PL.nPrincipal = 0) AS slideres

,(SELECT STRING_AGG(VLP.nlider,'','') FROM v_Listado_Proyectos_con_requerimientos VLP WHERE VLP.nId_Proyecto = V.nId_Proyecto) AS snidlideres

FROM v_Listado_Proyectos_con_requerimientos V WHERE ' + @conditionsClause + ' AND ' + @whereClause + ' ' + @orderClause + ' ' + @paginationClause

END

-- Ejecutar la consulta

EXEC sp_executesql @script

  

END