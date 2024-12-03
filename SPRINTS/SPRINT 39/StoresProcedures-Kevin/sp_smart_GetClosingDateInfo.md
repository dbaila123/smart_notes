```SQL
CREATE PROCEDURE sp_smart_GetClosingDateInfo

AS

BEGIN

SET NOCOUNT ON;

BEGIN TRY

  

DECLARE @dFecha_Actual DATE = CONVERT(DATE, GETDATE());

  

WITH RegistrosClosing AS (

SELECT TOP 1

'Vigente' as sTipoFecha,

nIdClosingDate,

nCantidadDias,

dSinceEffectDay,

nEstadoDate

FROM ClosingDate

WHERE dSinceEffectDay <= @dFecha_Actual

AND nEstadoDate = 1

ORDER BY dSinceEffectDay DESC -- Obtiene el más cercano a la fecha actual

  

UNION ALL

  

-- Obtener el próximo registro

SELECT TOP 1

'Próxima' as sTipoFecha,

nIdClosingDate,

nCantidadDias,

dSinceEffectDay,

nEstadoDate

FROM ClosingDate

WHERE dSinceEffectDay > @dFecha_Actual

AND nEstadoDate = 1

ORDER BY dSinceEffectDay ASC

)

SELECT

sTipoFecha,

nIdClosingDate,

nCantidadDias,

dSinceEffectDay,

nEstadoDate

FROM RegistrosClosing

ORDER BY dSinceEffectDay;

  

END TRY

BEGIN CATCH

SELECT

ERROR_MESSAGE() as Mensaje,

ERROR_LINE() as Linea,

ERROR_NUMBER() as Numero;

END CATCH

END;
```