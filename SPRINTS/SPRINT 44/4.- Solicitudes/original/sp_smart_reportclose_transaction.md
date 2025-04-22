```sql
CREATE PROCEDURE sp_smart_reportclose_transaction

@nId_Cierre INT

AS

BEGIN

/* SELECT *

FROM dbo.fn_get_request_hours_by_Transaction(null, 0, 0, 1,@nId_Cierre, NULL)*/

SELECT

tsm.nId_Colaborador,

p.sPersona_Nombre AS SNombreColaborador,

COUNT(CASE

WHEN tsm.nId_Sub_Tipo_Entidad = 6 AND tsm.nId_Tipo_Entidad = 1 THEN 1

END)AS Total_Solicitudes, -- Total de solicitudes solo para tipo 1

SUM(CASE

WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos

ELSE 0

END) AS CantidadTotalSuma, -- Sumar horas en contra 1

SUM(CASE

WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos

ELSE 0

END) / 60 AS CantidadTotalSuma_Horas, -- Sumar horas en contra 1

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalResta, -- Sumar horas a favor 2

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END) / 60 AS CantidadTotalResta_Horas,

CASE

WHEN (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END)) < 0

THEN 0

ELSE (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END))

END AS CantidadTotalDescontado, -- Total descontado

CASE

WHEN (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END)) < 0

THEN 0

ELSE (SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 THEN tsm.dCantidad_Minutos ELSE 0 END))

END / 60 AS CantidadTotalDescontado_Horas, -- Total descontado

CAST(MAX(tsm.dDatetime_Creacion) AS DATE) AS FechaUltimaTransaccion,

c.sUrl_Foto,

STRING_AGG(CAST(tsm.nId_Transaccion AS VARCHAR(MAX)), ',') AS sTransaccionesIds

FROM

Transacciones_Saldo_Mins tsm

JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador

JOIN Personas p ON p.nId_Persona = c.nId_Persona

WHERE

tsm.nId_Cierre = @nId_Cierre

AND tsm.nId_Tipo_Entidad IN (1, 9, 5)

GROUP BY

tsm.nId_Colaborador,

p.sPersona_Nombre,

c.sUrl_Foto

  

END;
```