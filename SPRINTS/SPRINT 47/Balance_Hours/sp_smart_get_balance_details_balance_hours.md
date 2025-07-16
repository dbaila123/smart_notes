```sql
CREATE PROCEDURE sp_smart_get_balance_details_balance_hours(

@nId_Collaborator INT,

@sFilterEight VARCHAR(MAX),

@TotalRecords INT OUTPUT

)

AS

BEGIN

SET NOCOUNT ON;

  

DECLARE @script AS nvarchar(max)

DECLARE @countScript AS NVARCHAR(MAX)

DECLARE @params AS NVARCHAR(MAX)

SET @countScript = N'

SELECT @TotalCount = COUNT(*)

FROM (

SELECT

tsm.nId_Cierre AS nId_Entity

FROM

Transacciones_Saldo_Mins tsm

JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador

JOIN Personas p ON p.nId_Persona = c.nId_Persona

WHERE

tsm.nId_Cierre IN (' + @sFilterEight + ')

AND tsm.nId_Colaborador = ' + CAST(@nId_Collaborator AS VARCHAR) + '

AND tsm.nId_Tipo_Entidad IN (1, 9, 5)

GROUP BY

tsm.nId_Colaborador,

p.sPersona_Nombre,

tsm.nId_Cierre

) AS CountQuery'

SET @params = N'@TotalCount INT OUTPUT'

EXECUTE sp_executesql @countScript, @params, @TotalCount = @TotalRecords OUTPUT

SET @script = N'

SELECT

tsm.nId_Cierre AS nId_Entity,

''SALDO'' AS sEntity_Type,

COUNT(CASE

WHEN tsm.nId_Sub_Tipo_Entidad = 6 AND tsm.nId_Tipo_Entidad = 1 THEN 1

END)AS Total_Solicitudes, -- Total de solicitudes solo para tipo 1

SUM(CASE

WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos

ELSE 0

END) AS nMinutos_En_Contra, -- Sumar horas en contra 1

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END) AS nMinutos_A_Favor, -- Sumar horas a favor 2

CASE

WHEN (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END)) < 0

THEN 0

ELSE (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END))

END AS nMinutos_Descontados,

CASE

WHEN (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END)) > 0

THEN 0

ELSE (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END)

-SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END))

END AS nMinutos_Saldo

FROM

Transacciones_Saldo_Mins tsm

JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador

JOIN Personas p ON p.nId_Persona = c.nId_Persona

WHERE

tsm.nId_Cierre IN ('+@sFilterEight+')

AND tsm.nId_Colaborador = '+CAST(@nId_Collaborator AS VARCHAR)+'

AND tsm.nId_Tipo_Entidad IN (1, 9, 5)

GROUP BY

tsm.nId_Colaborador,

p.sPersona_Nombre,

tsm.nId_Cierre'

  

EXECUTE sp_executesql @script

  

END
```