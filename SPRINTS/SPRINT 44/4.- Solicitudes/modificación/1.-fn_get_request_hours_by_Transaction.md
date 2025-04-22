```sql
CREATE FUNCTION fn_get_request_hours_by_Transaction(

@sId_Collaborator NVARCHAR(MAX) = NULL,

@bOnlyPendingRequest BIT = 1,

@bShowNegativeDiff BIT = 0,

@bOnlyRequestEnding BIT = 1,

@nId_Cierre INT = NULL,

@dFecha_Corte

  

DATETIME = NULL

)

RETURNS TABLE

AS

RETURN

SELECT

tsm.nId_Colaborador,

p.sPersona_Nombre AS SNombreColaborador,

con.sDescripcion as sTipo_Documento,

d.sNumero_Documento,

CONCAT(

con.sDescripcion,

' ',

d.sNumero_Documento

) as sDocumento_Completo,

COUNT(CASE

WHEN

@bOnlyRequestEnding = 1 AND tsm.nId_Sub_Tipo_Entidad = 6 AND tsm.nId_Tipo_Entidad = 1 THEN 1

WHEN @bOnlyRequestEnding = 0 AND tsm.nId_Sub_Tipo_Entidad IN(6, 7) AND tsm.nId_Tipo_Entidad = 1 THEN 1

END)AS Total_Solicitudes,

-- Total de solicitudes solo para tipo 1

SUM(CASE

WHEN @bOnlyRequestEnding = 1 AND tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos

WHEN @bOnlyRequestEnding = 0 AND tsm.nId_Sub_Tipo_Entidad IN(6, 7) THEN tsm.dCantidad_Minutos

ELSE 0

END) AS CantidadTotalSuma, -- Sumar horas en contra 1

SUM(CASE

WHEN @bOnlyRequestEnding = 1 AND tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos

WHEN @bOnlyRequestEnding = 0 AND tsm.nId_Sub_Tipo_Entidad IN(6,7) THEN tsm.dCantidad_Minutos

ELSE 0

END) / 60 AS CantidadTotalSuma_Horas, -- Sumar horas en contra 1

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalResta, -- Sumar horas a favor 2

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END) / 60 AS CantidadTotalResta_Horas,

CASE

WHEN @bShowNegativeDiff = 1 THEN (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END))

ELSE

CASE

WHEN (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) - SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END)) < 0

THEN 0

ELSE (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) - SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END))

END

END AS CantidadTotalDescontado, -- Total descontado

CASE

WHEN @bShowNegativeDiff = 1 THEN (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END))

ELSE

CASE

WHEN (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END)) < 0

THEN 0

ELSE (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END))

END

END / 60 AS

CantidadTotalDescontado_Horas, -- Total descontado

CAST(MAX(tsm.dDatetime_Creacion) AS DATE) AS FechaUltimaTransaccion,

c.sUrl_Foto,

STRING_AGG(CAST(tsm.nId_Transaccion AS VARCHAR(MAX)), ',') AS sTransaccionesIds

FROM

  

  

Transacciones_Saldo_Mins tsm

JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador

JOIN Personas p ON p.nId_Persona = c.nId_Persona

LEFT JOIN Documentos d ON d.nId_Persona = p.nId_Persona

LEFT JOIN Configs

con ON d.nTipo_Documento = con.sCodigo

AND con.sTabla = 'TIPO_DOCUMENTO'

WHERE

((@nId_Cierre IS NULL AND tsm.nId_Cierre IS NULL) OR (@nId_Cierre IS NOT NULL AND tsm.nId_Cierre = @nId_Cierre))

AND tsm.nId_Tipo_Entidad

IN (1, 9, 5)

AND (@sId_Collaborator IS NULL OR tsm.nId_Colaborador IN (SELECT value FROM STRING_SPLIT(@sId_Collaborator, ',')))

AND (@dFecha_Corte IS NULL OR (@dFecha_Corte IS NOT NULL AND CONVERT(DATE, tsm.dDatetime_Creacion) <= CONVERT(DATE,@dFecha_Corte)))

AND c.nEstado_Colaborador = 1

AND tsm.nEstado_Entidad = 3

GROUP BY

tsm.nId_Colaborador,

p.sPersona_Nombre,

c.sUrl_Foto,

con.sDescripcion,

d.sNumero_Documento

HAVING

0 < (CASE

WHEN @bOnlyPendingRequest = 1 THEN

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END)

ELSE 1

END)
```