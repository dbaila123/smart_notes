```sql
CREATE PROCEDURE sp_smart_get_request_details_balance_hours(

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

FROM Solicitudes s

WHERE s.nId_Colaborador = ' + CAST(@nId_Collaborator AS VARCHAR) + '

AND s.nId_Solictud IN (' + @sFilterEight + ')'

SET @params = N'@TotalCount INT OUTPUT'

EXECUTE sp_executesql @countScript, @params, @TotalCount = @TotalRecords OUTPUT

SET @script = N'

DECLARE @aSupervisors TABLE (nId_Supervisor INT, nId_User_Supervisor INT, sNombre_Supervisor NVARCHAR(MAX), sEmail_Supervisor NVARCHAR(MAX), nIsFinalApprover INT, nOrden INT)

  

INSERT INTO @aSupervisors

EXEC dbo.sp_Approver_Team_Get_Name_Email_UserId ' + CAST(@nId_Collaborator AS VARCHAR)+'

  

SELECT s.nId_Solictud AS nId_Entity,

s.nId_Tipo_Solicitud AS nEntity_Type,

ts.sDescripcion AS sEntity_Type,

s.nCantidad,

CONCAT(s.dFecha_Inicio,'' '', s.dHora_Inicio) AS dFecha_Inicio,

CONCAT(s.dFecha_Fin,'' '', s.dHora_Fin) AS dFecha_Fin,

s.dFecha_Solicitud AS dDate,

ts.nTipo_Frecuencia AS nFrecuency,

ts.nId_Categoria AS nCategory,

s.nEstado_Solicitud AS nEntity_State

FROM Solicitudes s

JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud

JOIN Categorias_Permisos cp ON cp.nId_Categoria = ts.nId_Categoria

JOIN Tipos_Motivos tm ON tm.nId_Tipo_Motivo = s.nId_Tipo_Motivo

WHERE s.nId_Colaborador = '+CAST(@nId_Collaborator AS VARCHAR) +'AND s.nId_Solictud in ('+@sFilterEight+')

'

EXECUTE sp_executesql @script

END
```