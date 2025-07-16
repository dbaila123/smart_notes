```sql
CREATE PROCEDURE sp_smart_get_attendance_details_balance_hours(

@nId_Collaborator INT,

@sFilterEight VARCHAR(MAX),

@TotalRecords INT OUTPUT

)

AS

BEGIN

  

DECLARE @script AS nvarchar(max)

DECLARE @countScript AS NVARCHAR(MAX)

DECLARE @params AS NVARCHAR(MAX)

SET @countScript = N'

SELECT @TotalCount = COUNT(*)

FROM Asistencias a

WHERE a.nId_Asistencia IN (' + @sFilterEight + ')

AND a.nId_Colaborador = ' + CAST(@nId_Collaborator AS VARCHAR)

SET @params = N'@TotalCount INT OUTPUT'

EXECUTE sp_executesql @countScript, @params, @TotalCount = @TotalRecords OUTPUT

SET @script = N'

SELECT

a.nId_Asistencia AS nId_Entity,

'+'''ASISTENCIA'''+' AS sEntity_Type,

a.dTiempo_Tardanza AS nCantidad,

A.dFecha_Asistencia AS dDate,

(

SELECT

m.nId_Marca,

m.dFecha_Marca AS dTiempo_Marca,

m.nTipo_Marcacion,

m.dFecha_Jornada as dFecha_Marca

FROM Marcas m

WHERE a.dFecha_Asistencia = m.dFecha_Jornada and a.nId_Colaborador = m.nId_Colaborador

FOR JSON PATH

) AS Marks

FROM Asistencias a

where a.nId_Asistencia in ('+@sFilterEight+') and a.nId_Colaborador = '+CAST(@nId_Collaborator AS VARCHAR)

  

EXECUTE sp_executesql @script

  

END
```