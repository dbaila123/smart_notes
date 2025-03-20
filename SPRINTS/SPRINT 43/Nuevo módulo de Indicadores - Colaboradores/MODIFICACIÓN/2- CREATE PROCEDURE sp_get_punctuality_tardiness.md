```SQL
CREATE PROCEDURE [dbo].[sp_get_punctuality_tardiness]

(

@nId_User INT,

@sSlugToGetList NVARCHAR(MAX),

@FechaInicio DATE,

@FechaFin DATE,

@sNombreColaborador NVARCHAR(MAX) = NULL,

@sFilterOne NVARCHAR(MAX) = NULL, -- Supervisores

@sFilterTwo NVARCHAR(MAX) = NULL, -- Estado del colaborador

@sId_Collaborator NVARCHAR(MAX) = NULL

)

AS

BEGIN

SET NOCOUNT ON;

  

-- Variables para filtros dinámicos

DECLARE @whereClause AS NVARCHAR(MAX) = '';

DECLARE @viewClause AS NVARCHAR(MAX) = ''; -- Nueva variable para la cláusula de vista

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

INSERT INTO @conditions VALUES ('a.nId_Colaborador IN (' + @CollaboratorsList + ')');

END

  

-- Filtro por nombre de colaborador

IF @sNombreColaborador IS NOT NULL

BEGIN

EXEC sp_get_like_condition 'p.sPersona_Nombre', @sNombreColaborador, @conditionOUT OUTPUT;

INSERT INTO @conditions VALUES (@conditionOUT);

END

  

IF @sFilterOne IS NOT NULL AND @sSlugToGetList <> 'INDICADORES-MODULO-COLABORADORES-EQUIPOS'

BEGIN

DECLARE @CollaboratorsListTeam VARCHAR(MAX);

SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne), 0, 0);

INSERT INTO @conditions VALUES ('a.nId_Colaborador IN (' + @CollaboratorsListTeam + ')');

END

-- Filtro por estado del colaborador

IF @sFilterTwo IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES ('c.nEstado_Colaborador IN (' + @sFilterTwo + ')');

END

-- Filtro por ID de colaborador específico

IF @sId_Collaborator IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES ('a.nId_Colaborador = ' + @sId_Collaborator + '');

END

  

-- Construcción de la cláusula WHERE dinámica

SELECT @count = COUNT(*) FROM @conditions;

IF @count > 0

BEGIN

SET @whereClause = N'AND ';

END

WHILE @count > 0

BEGIN

DECLARE @condition_temp NVARCHAR(MAX) = (SELECT TOP 1 condition FROM @conditions);

IF @count = 1

SET @whereClause = CONCAT(@whereClause, @condition_temp);

ELSE

SET @whereClause = CONCAT(@whereClause, @condition_temp, ' AND ');

DELETE TOP (1) FROM @conditions;

SELECT @count = COUNT(*) FROM @conditions;

END

  

-- Consulta principal

DECLARE @sqlQuery NVARCHAR(MAX);

SET @sqlQuery = N'

SELECT

a.nId_Colaborador,

p.sPersona_Nombre AS sNombres_Apellidos,

p.sPrimer_Nombre + '' '' + p.sApe_Paterno AS sNombre_Colaborador,

c.nEstado_Colaborador,

c3.sDescripcion,

@FechaInicio AS dFecha_Inicio,

@FechaFin AS dFecha_Fin,

COUNT(*) AS nTotal_Asistencias,

SUM(CASE WHEN c2.sDescripcion = ''PUNTUAL'' THEN 1 ELSE 0 END) AS nTotal_Puntualidad,

SUM(CASE WHEN c2.sDescripcion = ''TARDANZA'' THEN 1 ELSE 0 END) AS nTotal_Tardanza,

SUM(CASE WHEN c2.sDescripcion = ''EN TOLERANCIA'' THEN 1 ELSE 0 END) AS nTotal_En_Tolerancia,

-- Cálculo de porcentajes

CAST(SUM(CASE WHEN c2.sDescripcion = ''PUNTUAL'' THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS nPorcentaje_Puntualidad,

CAST(SUM(CASE WHEN c2.sDescripcion = ''TARDANZA'' THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS nPorcentaje_Tardanza,

CAST(SUM(CASE WHEN c2.sDescripcion = ''EN TOLERANCIA'' THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS nPorcentaje_En_Tolerancia

FROM Asistencias a

JOIN Colaboradores c ON c.nId_Colaborador = a.nId_Colaborador

JOIN Personas p ON p.nId_Persona = c.nId_Persona

LEFT JOIN Configs c2 ON c2.sCodigo = CONVERT(NVARCHAR(MAX), a.nEstado_Asistencia)

AND c2.sTabla = ''ESTADO_ASISTENCIA''

LEFT JOIN Configs c3 ON c3.sCodigo = CONVERT(NVARCHAR(MAX), c.nEstado_Colaborador)

AND c3.sTabla = ''ESTADO_COLABORADOR''

WHERE

a.dFecha_Asistencia BETWEEN @FechaInicio AND @FechaFin

' + @whereClause + '

AND c2.sDescripcion IN (''PUNTUAL'', ''EN TOLERANCIA'', ''TARDANZA'')

GROUP BY a.nId_Colaborador, p.sPersona_Nombre,p.sPrimer_Nombre, p.sApe_Paterno,c.nEstado_Colaborador,c3.sDescripcion

ORDER BY nPorcentaje_Puntualidad DESC;'

  

-- Ejecutar la consulta dinámica

EXEC sp_executesql

@sqlQuery,

N'@FechaInicio DATE, @FechaFin DATE',

@FechaInicio = @FechaInicio,

@FechaFin = @FechaFin;

END;