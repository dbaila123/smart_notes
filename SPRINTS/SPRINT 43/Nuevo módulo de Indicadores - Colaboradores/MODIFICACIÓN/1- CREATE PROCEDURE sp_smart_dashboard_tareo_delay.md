```SQL
CREATE PROCEDURE sp_smart_dashboard_tareo_delay (

@nId_User INT,

@sSlugToGetList NVARCHAR(MAX),

@FechaInicio DATE,

@FechaFin DATE,

@sNombreColaborador VARCHAR(MAX) = NULL,

@sFilterOne NVARCHAR(MAX) = NULL, -- Supervisores

@sfilterTwo NVARCHAR(MAX) = NULL -- Estado

)

AS

BEGIN

SET NOCOUNT ON;

  

-- Variables para el filtro dinámico

DECLARE @whereClause AS NVARCHAR(MAX) = '';

DECLARE @viewClause AS NVARCHAR(MAX) = '';

DECLARE @conditions AS TABLE (condition NVARCHAR(MAX));

DECLARE @conditionOUT AS NVARCHAR(MAX);

DECLARE @count INT;

  

IF @sSlugToGetList = 'INDICADORES-MODULO-COLABORADORES'

BEGIN

SET @viewClause = N'';

END

ELSE IF @sSlugToGetList = 'INDICADORES-MODULO-COLABORADORES-EQUIPOS'

BEGIN

DECLARE @CollaboratorsList VARCHAR(MAX);

DECLARE @nId_Coll INT = (SELECT c.nId_Colaborador FROM Colaboradores c

JOIN Usuarios u ON u.nId_Persona = c.nId_Persona

WHERE u.nId_Usuario = @nId_User);

SET @CollaboratorsList = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @nId_Coll), 0, 0);

INSERT INTO @conditions VALUES ('v.nId_Colaborador IN (' + @CollaboratorsList + ')');

END

-- Convertir @SNombreColaborador a minúsculas

SET @sNombreColaborador = LOWER(@sNombreColaborador);

  

-- Construir condición de búsqueda por nombre

IF @sNombreColaborador IS NOT NULL

BEGIN

EXEC sp_get_multiple_like_condition 'sNombre_Completo', @sNombreColaborador, @conditionOUT OUTPUT;

INSERT INTO @conditions VALUES (@conditionOUT);

END

  

IF @sFilterOne IS NOT NULL AND @sSlugToGetList <> 'INDICADORES-MODULO-COLABORADORES-EQUIPOS'

BEGIN

DECLARE @CollaboratorsListTeam VARCHAR(MAX);

SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne), 0, 0);

INSERT INTO @conditions VALUES ('v.nId_Colaborador IN (' + @CollaboratorsListTeam + ')');

END

-- Construir condición de búsqueda por estado del colaborador

IF @sfilterTwo IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES('v.nEstado_Colaborador IN ('+@sfilterTwo+')')

END

  

-- Construir WHERE clause

SELECT @count = COUNT(*) FROM @conditions;

  

IF @count > 0

SET @whereClause = N' AND ';

  

WHILE @count > 0

BEGIN

DECLARE @condition_temp VARCHAR(MAX) = (SELECT TOP(1) condition FROM @conditions);

  

IF @count = 1

SET @whereClause = CONCAT(@whereClause, @condition_temp);

ELSE

SET @whereClause = CONCAT(@whereClause, @condition_temp, ' AND ');

  

DELETE TOP(1) FROM @conditions;

SELECT @count = COUNT(*) FROM @conditions;

END

  

CREATE TABLE #UltimosRegistrosTareo (

nId_Colaborador INT,

NombreColaborador NVARCHAR(200),

NombreCompleto NVARCHAR(200),

FechaRegistro DATE,

FechaCreacion DATETIME,

FechaLimite DATE,

DiasRetraso INT,

EstadoTareo VARCHAR(50),

EstadoColaborador INT

);

  

DECLARE @sql NVARCHAR(MAX) = N'

WITH UltimosTareos AS (

SELECT

v.nId_Colaborador,

v.sNombre_Colaborador,

v.sNombre_Completo,

v.dFecha_Registro,

v.dDatetime_Creador,

v.dSiguiente_DiaHabil,

v.nEstado_Colaborador,

ROW_NUMBER() OVER (

PARTITION BY v.nId_Colaborador, v.dFecha_Registro

ORDER BY v.dDatetime_Creador DESC

) AS RN

FROM

v_Listado_TareoStatus v

WHERE

v.dFecha_Registro BETWEEN @FechaInicio AND @FechaFin' + @whereClause + '

)

INSERT INTO #UltimosRegistrosTareo

SELECT

ut.nId_Colaborador,

ut.sNombre_Colaborador,

ut.sNombre_Completo,

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

END AS EstadoTareo,

ut.nEstado_Colaborador

FROM

UltimosTareos ut

WHERE

RN = 1';

  

-- Ejecutar la consulta dinámica

EXEC sp_executesql @sql, N'@FechaInicio DATE, @FechaFin DATE', @FechaInicio, @FechaFin;

  

CREATE TABLE #DiasLaborados (

nId_Colaborador INT,

nDias_Laborados INT

);

  

INSERT INTO #DiasLaborados

SELECT

a.nId_Colaborador,

COUNT(DISTINCT a.dFecha_Asistencia) AS nDias_Laborados

FROM Asistencias a

WHERE a.dFecha_Asistencia BETWEEN @FechaInicio AND @FechaFin

AND a.nTiene_Marca >= 1

--AND a.nLlenado > 0

AND sEstadoLLenadoTareo IN ('COMPLETO','INCOMPLETO')

GROUP BY a.nId_Colaborador;

  

SELECT

t.nId_Colaborador,

t.NombreColaborador AS sNombre_Colaborador,

t.NombreCompleto AS sNombre_Completo,

dl.nDias_Laborados,

@FechaInicio AS dFecha_Inicio,

@FechaFin AS dFecha_fin,

COUNT(CASE WHEN t.EstadoTareo = 'Retrasado' THEN 1 END) AS nDias_Tareo_Retraso,

--porcentaje de cumplimiento

CASE

WHEN ISNULL(dl.nDias_Laborados, 0) > 0 THEN

CAST(((CAST(dl.nDias_Laborados AS DECIMAL(10, 2)) -

COUNT(CASE WHEN t.EstadoTareo = 'Retrasado' THEN 1 END)) * 100.0 /

CAST(dl.nDias_Laborados AS DECIMAL(10, 2))) AS DECIMAL(10, 2))

ELSE 0

END AS nPorcentaje_Cumplimiento,

t.EstadoColaborador as nEstado_Colaborador

FROM

#UltimosRegistrosTareo t

INNER JOIN #DiasLaborados dl ON t.nId_Colaborador = dl.nId_Colaborador

GROUP BY

t.nId_Colaborador,

t.NombreColaborador,

t.NombreCompleto,

dl.nDias_Laborados,

t.EstadoColaborador

ORDER BY

nDias_Tareo_Retraso DESC;

  

DROP TABLE #UltimosRegistrosTareo;

DROP TABLE #DiasLaborados;

END;