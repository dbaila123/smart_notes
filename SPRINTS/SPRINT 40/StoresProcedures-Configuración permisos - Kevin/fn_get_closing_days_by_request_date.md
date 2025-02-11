```SQL
CREATE FUNCTION fn_get_closing_days_by_request_date

(

@fecha_solicitud DATE

)

RETURNS INT

AS

BEGIN

DECLARE @dias_cierre INT;

SELECT TOP 1 @dias_cierre = nCantidadDias

FROM ClosingDate

WHERE dSinceEffectDay <= @fecha_solicitud

AND nEstadoDate = 1

ORDER BY dSinceEffectDay DESC;

RETURN (@dias_cierre);

END;
```