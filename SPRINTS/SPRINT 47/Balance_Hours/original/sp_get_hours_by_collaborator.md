```sql
CREATE PROCEDURE [dbo].[sp_get_hours_by_collaborator](

@sTipo_listado NVARCHAR(MAX) = NULL,

@nId_User INT,

@dFecha DATETIME,

@sIds_Colaborador NVARCHAR(MAX) = NULL,

@sFilterOne NVARCHAR(MAX) = NULL,--Filtro equipo

@sFilterTwo NVARCHAR(MAX) = NULL, --Filtro estado colaborador

@sFilterThree NVARCHAR(MAX) = '1', --Filtro estado horas

@sTextFilter NVARCHAR(MAX) = NULL,

@bPendiente BIT = 1

) AS

BEGIN

DECLARE @script AS nvarchar(max)

DECLARE @whereClause AS nvarchar(max) = ''

DECLARE @conditions AS TABLE (condition nvarchar(max))

DECLARE @dFecha_cierre varchar(20)

DECLARE @conditionOUT AS nvarchar(max);

DECLARE @nId_Colaborador INT = (SELECT c.nId_Colaborador FROM Colaboradores c

JOIN Personas p ON p.nId_Persona = c.nId_Persona

JOIN Usuarios u ON u.nId_Persona = p.nId_Persona

WHERE u.nId_Usuario = @nId_User)

  

SELECT TOP 1 @dFecha_cierre = dFecha_cierre from closure_request order by nid_cierre desc

  

--a�adir slug equipos

IF @sTipo_listado = 'INDICADORES-MODULO-COLABORADORES-EQUIPOS'

BEGIN

DECLARE @ListTeam VARCHAR(MAX);

SET @ListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @nId_Colaborador),0,0)

INSERT INTO @conditions VALUES(N't.nId_Colaborador in('+@ListTeam+')')

END

  

IF @sFilterOne IS NOT NULL

BEGIN

DECLARE @CollaboratorsListTeam VARCHAR(MAX);

SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)

INSERT INTO @conditions VALUES(N't.nId_Colaborador in('+@CollaboratorsListTeam+')')

END

  

IF @sFilterTwo IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES ('c.nEstado_Colaborador IN (' + @sFilterTwo + ')');

END

  

IF @sTextFilter IS NOT NULL

BEGIN

EXEC sp_get_like_condition 'p.sPersona_Nombre', @sTextFilter, @conditionOUT OUTPUT

INSERT INTO @conditions VALUES (@conditionOUT);

END

  

IF @sIds_Colaborador IS NOT NULL

BEGIN

INSERT INTO @conditions VALUES(N't.nId_Colaborador in('+ @sIds_Colaborador +')')

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

END

  

--Principal Query

SET @script = N'

SELECT

t.nId_Colaborador,

p.sPersona_Nombre as sNombre_Colaborador,

t.dTardanza_Con_Tolerancia,

t.nTotal_Min_No_Recuperables AS dTardanza_Sin_Tolerancia,

CASE

WHEN ' + @sFilterThree + ' = ''1'' THEN t.nTotal_Min_Contra_Per_Apro

WHEN ' + @sFilterThree + ' = ''3'' THEN t.nPrePlanilla_Tiempo_Permisos

ELSE t.nTotal_Min_Contra_Per_AP

END AS nPermisos,

CASE

WHEN ' + @sFilterThree + ' = ''1'' OR ' + @sFilterThree + ' = ''3'' THEN t.nTotal_Min_Favor_CompRecu_Apro + t.nTotal_Bolsa_Cierre

ELSE t.nTotal_Min_Favor_CompRecu_AP + t.nTotal_Bolsa_Cierre

END AS nAFavor,

CASE

WHEN ' + @sFilterThree + ' = ''1'' THEN t.nTotal_Min_Contra_Per_Apro + t.dTardanza_Con_Tolerancia + t.nTotal_Min_No_Recuperables

WHEN ' + @sFilterThree + ' = ''3'' THEN t.nPrePlanilla_Tiempo_Permisos + t.dTardanza_Con_Tolerancia + t.nTotal_Min_No_Recuperables

ELSE t.nTotal_Min_Contra_Per_AP + t.dTardanza_Con_Tolerancia + t.nTotal_Min_No_Recuperables

END AS nEnContra,

CASE

WHEN ' + @sFilterThree + ' = ''1'' OR ' + @sFilterThree + ' = ''3'' THEN t.nTotal_Min_Favor_Extra_Apro

ELSE t.nTotal_Min_Favor_Extra_AP

END AS nExtra,

' + CAST(@dFecha_cierre AS VARCHAR) + ' AS dFecha_cierre,

CASE

WHEN ' + @sFilterThree + ' = ''1'' OR ' + @sFilterThree + ' = ''3'' THEN t.nTotal_Min_Favor_CompRecu_Apro

ELSE t.nTotal_Min_Favor_CompRecu_AP

END AS dCompensacion_Minutos,

t.nTotal_Bolsa_Cierre AS dBolsa_Cierre

FROM Total_Min_Apro_Pend t

JOIN Colaboradores c on t.nId_Colaborador = c.nId_Colaborador

JOIN Personas p on c.nId_Persona = p.nId_Persona

'+ @whereClause +'

ORDER BY nId_Colaborador

'

  

  

EXECUTE sp_executesql @script

END
```