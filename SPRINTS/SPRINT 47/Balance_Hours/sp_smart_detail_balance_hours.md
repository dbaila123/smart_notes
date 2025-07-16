```sql
CREATE PROCEDURE sp_smart_detail_balance_hours

(

@sTipo_listado NVARCHAR(MAX) = NULL,

@nId_User INT,

@dFecha NVARCHAR(MAX),

@sFilterOne NVARCHAR(MAX) = NULL,--Filtro equipo

@sFilterTwo NVARCHAR(MAX) = NULL, --Filtro estado colaborador

@sFilterThree NVARCHAR(MAX) = '1', --Filtro estado horas

@sTextFilter NVARCHAR(MAX) = NULL,

@bPendiente BIT = 1,

@pnum INT = NULL, --Número de Páginas

@size INT = NULL --Cantidad de Registros por página

)

AS

BEGIN

DECLARE @script AS nvarchar(max)

DECLARE @whereClause AS nvarchar(max) = ''

DECLARE @conditions AS TABLE (condition nvarchar(max))

DECLARE @dFecha_cierre varchar(20)

DECLARE @paginationClause AS nvarchar(max)

DECLARE @conditionOUT AS nvarchar(max);

DECLARE @dFecha_Cierre_Planificado DATE;

DECLARE @nClosing_Days INT;

DECLARE @nId_Colaborador INT = (SELECT c.nId_Colaborador FROM Colaboradores c

JOIN Personas p ON p.nId_Persona = c.nId_Persona

JOIN Usuarios u ON u.nId_Persona = p.nId_Persona

WHERE u.nId_Usuario = @nId_User)

--añadir slug equipos

IF @sTipo_listado = 'INDICADORES-MODULO-COLABORADORES-EQUIPOS'

BEGIN

DECLARE @ListTeam VARCHAR(MAX);

SET @ListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @nId_Colaborador),0,0)

INSERT INTO @conditions VALUES(N'nId_Colaborador in('+@ListTeam+')')

END

  

IF @sFilterOne IS NOT NULL

BEGIN

DECLARE @CollaboratorsListTeam VARCHAR(MAX);

SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)

INSERT INTO @conditions VALUES(N'nId_Colaborador in('+@CollaboratorsListTeam+')')

END

  

IF @sFilterTwo IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES ('nEstado_Colaborador IN (' + @sFilterTwo + ')');

END

  

--FILTRO PRE PLANILLA

IF @sFilterThree = '3'

BEGIN

SELECT TOP 1

@dFecha_Cierre_Planificado = car.dFecha_Cierre_Planificado

FROM cierreAsistenciaReportes car

ORDER BY car.nId_Cierre desc

END

  

IF @sTextFilter IS NOT NULL

BEGIN

EXEC sp_get_like_condition 'sNombre_Colaborador', @sTextFilter, @conditionOUT OUTPUT

INSERT INTO @conditions VALUES (@conditionOUT);

END

  

--Agregar condicionales

DECLARE @count INT

  

SELECT @count = COUNT(*) FROM @conditions

IF @count > 0

BEGIN

SET @whereClause = N'WHERE '

END

  

WHILE @count > 0

BEGIN

DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)

IF @count = 1

BEGIN

SET @whereClause = concat (@whereClause,@condition_temp)

END

ELSE

BEGIN

SET @whereClause = concat (@whereClause,@condition_temp,' and ')

END

DELETE TOP (1) FROM @conditions

SELECT @count = COUNT(*) FROM @conditions

END;

  

--PAGINATION

IF @pnum IS NULL AND @size IS NULL

BEGIN

SET @paginationClause = ''

END

ELSE

BEGIN

EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT

END;

  

--query

SELECT

tsm.nId_Colaborador,

STRING_AGG(tsm.nId_Entidad, '-') AS sId_Cierre,

CAST(SUM(tsm.dCantidad_Minutos) AS INT) AS sTiempo

INTO #Bolsa

FROM transacciones_saldo_mins tsm

WHERE tsm.nId_Tipo_Entidad = 5

AND tsm.nId_Sub_Tipo_Entidad = 8

AND tsm.nId_Cierre IS NULL

AND CONVERT(DATE, tsm.dDatetime_Creacion) <= @dFecha

GROUP BY

tsm.nId_Colaborador,

tsm.nId_Entidad

  

  

SELECT

t.nId_Colaborador,

STRING_AGG(s.nId_Solictud, '-') AS sSolicitudes,

CAST(SUM(nMinutos) AS INT) AS sTiempo

INTO #Extras

FROM Tareas t

JOIN Solicitudes s ON t.nId_Tarea = s.nId_Tarea and s.nEstado_Solicitud in (1, 4, 3)

WHERE t.sTipo_Hora = 2

AND (

((@sFilterThree = '1' OR @sFilterThree = '3') AND t.nEstado_Tarea = 3) OR

(@sFilterThree = '2' AND t.nEstado_Tarea IN (1, 3, 9)))

AND t.nEstado_Pago IS NULL

AND t.dFecha_Registro <= @dFecha

GROUP BY t.nId_Colaborador

SELECT

tsm.nId_Colaborador,

STRING_AGG(t.nId_Solictud, '-') AS sSolicitudes,

CAST(SUM(tsm.dCantidad_Minutos) AS INT) AS sTiempo

INTO #Compensacion

FROM Transacciones_Saldo_Mins tsm

JOIN Solicitudes t ON tsm.nId_Entidad = t.nId_Tarea

WHERE tsm.nId_Tipo_Entidad = 9

AND t.nId_Tipo_Solicitud = 7

AND tsm.nId_Sub_Tipo_Entidad = 8

AND tsm.nId_Cierre IS NULL

AND (

((@sFilterThree = '1' OR @sFilterThree = '3') AND t.nEstado_Solicitud = 3) OR

(@sFilterThree = '2' AND t.nEstado_Solicitud IN (1, 3, 4)))

AND CONVERT(DATE, tsm.dDatetime_Creacion) <= @dFecha

GROUP BY tsm.nId_Colaborador

SELECT

tsm.nId_Colaborador,

STRING_AGG(t.nId_Solictud, '-') AS sSolicitudes,

CAST(SUM(tsm.dCantidad_Minutos) AS INT) AS sTiempo

INTO #Recuperacion

FROM Transacciones_Saldo_Mins tsm

JOIN Solicitudes t ON tsm.nId_Entidad = t.nId_Tarea

WHERE tsm.nId_Tipo_Entidad = 9

AND t.nId_Tipo_Solicitud = 8

AND tsm.nId_Sub_Tipo_Entidad = 8

AND tsm.nId_Cierre IS NULL

AND (

((@sFilterThree = '1' OR @sFilterThree = '3') AND t.nEstado_Solicitud = 3) OR

(@sFilterThree = '2' AND t.nEstado_Solicitud IN (1, 3, 4)))

AND CONVERT(DATE, tsm.dDatetime_Creacion) <= @dFecha

GROUP BY tsm.nId_Colaborador

SELECT

t.nId_Colaborador,

STRING_AGG(t.nId_Solicitud, '-') AS sSolicitudes,

CAST(SUM(nPermisos) AS INT) AS sTiempo

INTO #Permiso

FROM (

SELECT

tsm.nId_Colaborador,

tsm.nId_Entidad AS nId_Solicitud,

SUM(CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad IN (6,7)

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) AS nPermisos

FROM Transacciones_Saldo_Mins tsm

JOIN Solicitudes st ON tsm.nId_Entidad = st.nId_Solictud

WHERE tsm.nEstado_Transaccion = 1

AND tsm.nId_Tipo_Entidad = 1

AND (

((@sFilterThree = '1' OR @sFilterThree = '3') AND st.nEstado_Solicitud = 3) OR

(@sFilterThree = '2' AND st.nEstado_Solicitud IN (1, 3, 4)))

AND CONVERT(DATE, tsm.dDatetime_Creacion) <= @dFecha

AND (

(

@sFilterThree IS NULL OR @sFilterThree <> '3'

) OR

DATEADD(DAY, dbo.fn_get_closing_days_by_request_date(CAST(tsm.dDatetime_Creacion AS DATE)), tsm.dDatetime_Creacion) <= @dFecha_Cierre_Planificado

)

GROUP BY tsm.nId_Colaborador, tsm.nId_Entidad, tsm.dDatetime_Creacion, st.nEstado_Solicitud

HAVING SUM(

CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad IN (6,7)

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) > 0

) t GROUP BY t.nId_Colaborador

  

SELECT

t.nId_Colaborador,

STRING_AGG(t.nId_Entidad, '-') AS sAsistencia,

CAST(SUM(t.dTardanza_Con_Tolerancia) AS INT) AS sTiempo

INTO #Tolerancia

FROM (

SELECT

tsm.nId_Colaborador,

SUM(CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad = 3

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos ELSE 0

END) -

SUM(CASE

WHEN tsm.nTipo_Transaccion = 2

AND tsm.nId_Sub_Tipo_Entidad = 3

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) AS dTardanza_Con_Tolerancia,

tsm.nId_Entidad

FROM Transacciones_Saldo_Mins tsm

LEFT JOIN

Asistencias a ON a.nId_Asistencia = tsm.nId_Entidad

WHERE tsm.nEstado_Transaccion = 1

AND tsm.nId_Tipo_Entidad = 2

AND CONVERT(DATE, a.dFecha_Asistencia) <= @dFecha

GROUP BY tsm.nId_Colaborador, tsm.nId_Entidad, a.dFecha_Asistencia

HAVING SUM(CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad = 3

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) -

SUM(CASE

WHEN tsm.nTipo_Transaccion = 2

AND tsm.nId_Sub_Tipo_Entidad = 3

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) > 0

) t

GROUP BY t.nId_Colaborador

  

SELECT

t.nId_Colaborador,

STRING_AGG(t.nId_Entidad, '-') AS sAsistencia,

CAST(SUM(t.dTardanza_Sin_Tolerancia) AS INT) AS sTiempo

INTO #Tardanza_Sin_Tolerancia

FROM (

SELECT

tsm.nId_Colaborador,

SUM(CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad = 4

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) -

SUM(CASE

WHEN tsm.nTipo_Transaccion = 2

AND tsm.nId_Sub_Tipo_Entidad = 4

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) AS dTardanza_Sin_Tolerancia,

tsm.nId_Entidad

FROM Transacciones_Saldo_Mins tsm LEFT JOIN Asistencias a ON a.nId_Asistencia = tsm.nId_Entidad

WHERE

tsm.nEstado_Transaccion = 1

AND tsm.nId_Tipo_Entidad = 2

AND CONVERT(DATE, a.dFecha_Asistencia) <= @dFecha

GROUP BY tsm.nId_Colaborador, tsm.nId_Entidad, a.dFecha_Asistencia

HAVING

SUM(CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad = 4

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) -

SUM(CASE

WHEN tsm.nTipo_Transaccion = 2

AND tsm.nId_Sub_Tipo_Entidad = 4

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) > 0

) t

GROUP BY t.nId_Colaborador;

WITH Colaboradores_Unicos AS(

SELECT nId_Colaborador

FROM #Bolsa

UNION

SELECT nId_Colaborador

FROM #Extras

UNION

SELECT nId_Colaborador

FROM #Compensacion

UNION

SELECT nId_Colaborador

FROM #Recuperacion

UNION

SELECT nId_Colaborador

FROM #Permiso

UNION

SELECT nId_Colaborador

FROM #Tolerancia

UNION

SELECT nId_Colaborador

FROM #Tardanza_Sin_Tolerancia

)

SELECT

cu.nId_Colaborador,

c.sUrl_Foto as sFoto,

c.nEstado_Colaborador,

pe.sPersona_Nombre as sNombre_Colaborador,

e.sSolicitudes as sId_Extras,

e.sTiempo as sExtras,

b.sId_Cierre as sId_Bolsa,

b.sTiempo as sBolsa,

co.sSolicitudes as sId_Compensacion,

co.sTiempo as sCompensacion,

r.sSolicitudes as sId_Recuperacion,

r.sTiempo as sRecuperacion,

p.sSolicitudes as sId_Permisos,

p.sTiempo as sPermisos,

t.sAsistencia as sId_Tardanza_Recuperable,

t.sTiempo as sTardanza_Recuperable,

tst.sAsistencia as sId_Tardanza_No_Recuperable,

tst.sTiempo as sTardanza_No_Recuperable

INTO #Temporal

FROM Colaboradores_Unicos cu

JOIN Colaboradores c on cu.nId_Colaborador = c.nId_Colaborador

JOIN Personas pe on c.nId_Persona = pe.nId_Persona

LEFT JOIN #Extras e on cu.nId_Colaborador = e.nId_Colaborador

LEFT JOIN #Bolsa b on cu.nId_Colaborador = b.nId_Colaborador

LEFT JOIN #Compensacion co on cu.nId_Colaborador = co.nId_Colaborador

LEFT JOIN #Recuperacion r on cu.nId_Colaborador = r.nId_Colaborador

LEFT JOIN #Permiso p on cu.nId_Colaborador = p.nId_Colaborador

LEFT JOIN #Tolerancia t on cu.nId_Colaborador = t.nId_Colaborador

LEFT JOIN #Tardanza_Sin_Tolerancia tst on cu.nId_Colaborador = tst.nId_Colaborador

  

  

SET @script = N'

SELECT

nId_Colaborador,

sFoto,

sNombre_Colaborador,

sId_Extras,

sExtras,

sId_Bolsa,

sBolsa,

sId_Compensacion,

sCompensacion,

sId_Recuperacion,

sRecuperacion,

sId_Permisos,

sPermisos,

sId_Tardanza_Recuperable,

sTardanza_Recuperable,

sId_Tardanza_No_Recuperable,

sTardanza_No_Recuperable,

COUNT(*) OVER() AS nTotal

FROM #Temporal

' + @whereClause +'

ORDER BY nId_Colaborador

' + @paginationClause

  

EXECUTE sp_executesql @script

  

END
```