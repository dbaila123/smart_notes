```SQL
CREATE PROCEDURE sp_smart_dashboard_tareo_delay

@FechaInicio DATE,

@FechaFin DATE,

@sNombreColaborador varchar(MAX) = NULL,

@sFilterOne NVARCHAR(MAX) = NULL -- Supervisores

AS

BEGIN

SET NOCOUNT ON;

-- Variables para el filtro dinámico

DECLARE @whereClause AS nvarchar(max) = ''

DECLARE @conditions AS TABLE (condition nvarchar(max))

DECLARE @conditionOUT AS nvarchar(max)

DECLARE @count INT

  

-- Construir condición de búsqueda por nombre

IF @SNombreColaborador IS NOT NULL

BEGIN

EXEC sp_get_multiple_like_condition 'sNombre_Colaborador', @SNombreColaborador, @conditionOUT OUTPUT

INSERT INTO @conditions VALUES (@conditionOUT)

END

  

IF @sFilterOne IS NOT NULL

BEGIN

DECLARE @CollaboratorsListTeam VARCHAR(MAX);

SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)

INSERT INTO @conditions VALUES ('nId_Colaborador in('+@CollaboratorsListTeam+')')

END

  

  

-- Construir WHERE clause

SELECT @count = COUNT(*) FROM @conditions

IF @count > 0

BEGIN

SET @whereClause = N' AND '

END

  

WHILE @count > 0

BEGIN

DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)

IF @count = 1

BEGIN

SET @whereClause = CONCAT(@whereClause, @condition_temp)

END

ELSE

BEGIN

SET @whereClause = CONCAT(@whereClause, @condition_temp, ' AND ')

END

DELETE TOP(1) FROM @conditions

SELECT @count = COUNT(*) FROM @conditions

END

  

CREATE TABLE #UltimosRegistrosTareo (

nId_Colaborador INT,

NombreColaborador NVARCHAR(200),

FechaRegistro DATE,

FechaCreacion DATETIME,

FechaLimite DATE,

DiasRetraso INT,

EstadoTareo VARCHAR(50)

);

  

DECLARE @sql nvarchar(max) = N'

WITH UltimosTareos AS (

SELECT

v.nId_Colaborador,

v.sNombre_Colaborador,

v.dFecha_Registro,

v.dDatetime_Creador,

v.dSiguiente_DiaHabil,

ROW_NUMBER() OVER (

PARTITION BY v.nId_Colaborador, v.dFecha_Registro

ORDER BY v.dDatetime_Creador DESC

) AS RN

FROM

v_Listado_TareoStatus v

WHERE

v.dFecha_Registro BETWEEN @FechaInicio AND @FechaFin'

+ @whereClause + '

)

INSERT INTO #UltimosRegistrosTareo

SELECT

ut.nId_Colaborador,

ut.sNombre_Colaborador,

ut.dFecha_Registro,

ut.dDatetime_Creador,

ut.dSiguiente_DiaHabil AS FechaLimite,

dbo.fn_GetBusinessDays(

ut.dSiguiente_DiaHabil,

ISNULL(ut.dDatetime_Creador, GETDATE())

) AS DiasRetraso,

CASE

WHEN ut.dDatetime_Creador IS NULL THEN ''Pendiente''

WHEN CAST(ut.dDatetime_Creador AS DATE) <= ut.dSiguiente_DiaHabil THEN ''A Tiempo''

ELSE ''Retrasado''

END AS EstadoTareo

FROM

UltimosTareos ut

WHERE

RN = 1';

  

-- Ejecutar la consulta dinámica

EXEC sp_executesql @sql,

N'@FechaInicio DATE, @FechaFin DATE',

@FechaInicio, @FechaFin;

CREATE TABLE #DiasLaborados (

nId_Colaborador INT,

nDias_Laborados INT

);

  

INSERT INTO #DiasLaborados

SELECT

a.nId_Colaborador,

COUNT(DISTINCT a.dFecha_Asistencia) as nDias_Laborados

FROM Asistencias a

WHERE a.dFecha_Asistencia BETWEEN @FechaInicio AND @FechaFin

AND a.nTiene_Marca >= 1

GROUP BY a.nId_Colaborador;

  

SELECT

t.nId_Colaborador,

t.NombreColaborador as sNombre_Colaborador,

dl.nDias_Laborados,

COUNT(CASE WHEN t.EstadoTareo = 'Retrasado' THEN 1 END) AS nDias_Tareo_Retraso,

/* MAX(t.DiasRetraso) AS nMax_DiasRetraso,

COUNT(CASE

WHEN t.EstadoTareo = 'Retrasado' AND

t.DiasRetraso <= 2 THEN 1

END) AS nRetrasos_Leves,

COUNT(CASE

WHEN t.EstadoTareo = 'Retrasado' AND

t.DiasRetraso > 2 AND

t.DiasRetraso <= 5 THEN 1

END) AS nRetrasos_Moderados,

COUNT(CASE

WHEN t.EstadoTareo = 'Retrasado' AND

t.DiasRetraso > 5 THEN 1

END) AS nRetrasos_Graves,

SUM(CASE

WHEN t.FechaRegistro >= DATEADD(DAY, -7, @FechaFin)

AND t.EstadoTareo = 'Retrasado' THEN 1

ELSE 0

END) AS nRetrasos_UltimaSemana,*/

--porcentaje de cumplimiento

CASE

WHEN dl.nDias_Laborados > 0 THEN

CAST(((dl.nDias_Laborados - COUNT(CASE WHEN t.EstadoTareo = 'Retrasado' THEN 1 END)) * 100.0 / dl.nDias_Laborados) AS DECIMAL(5,2))

ELSE 0

END AS nPorcentaje_Cumplimiento

FROM

#UltimosRegistrosTareo t

LEFT JOIN #DiasLaborados dl ON t.nId_Colaborador = dl.nId_Colaborador

GROUP BY

t.nId_Colaborador,

t.NombreColaborador,

dl.nDias_Laborados

ORDER BY

nDias_Tareo_Retraso DESC;

  

DROP TABLE #UltimosRegistrosTareo;

DROP TABLE #DiasLaborados;

END;