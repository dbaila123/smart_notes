```SQL
CREATE PROCEDURE sp_smart_GetClosingDateInfo

AS

BEGIN

SET

NOCOUNT ON;

BEGIN

TRY

-- Declarar variable para la fecha actual

DECLARE @dFecha_Actual DATE = CONVERT(DATE,

GETDATE());

-- Obtener y devolver la fecha vigente y la próxima

SELECT

'Vigente' as sTipoFecha,

nIdClosingDate,

nCantidadDias,

dSinceEffectDay,

nEstadoDate

FROM

ClosingDate

WHERE

dSinceEffectDay <= @dFecha_Actual

AND nEstadoDate = 1

UNION ALL

  

SELECT

TOP 1

'Próxima' as sTipoFecha,

nIdClosingDate,

nCantidadDias,

dSinceEffectDay,

nEstadoDate

FROM

ClosingDate

WHERE

dSinceEffectDay > @dFecha_Actual

AND nEstadoDate = 2

ORDER BY

dSinceEffectDay;

END TRY

BEGIN CATCH

SELECT

ERROR_MESSAGE() as Mensaje,

ERROR_LINE() as Linea,

ERROR_NUMBER() as Numero

END CATCH

END;
```