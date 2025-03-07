```SQL
ALTER FUNCTION fn_GetNextBusinessDay (

@Date DATE

)

RETURNS DATE

AS

BEGIN

DECLARE @NextDay DATE = DATEADD(DAY, 1, @Date);

  

WHILE (1 = 1)

BEGIN

-- Verificar si es fin de semana o feriado

IF DATEPART(WEEKDAY, @NextDay) NOT IN (1, 7)

AND NOT EXISTS (

SELECT 1

FROM Feriados

WHERE dFecha = @NextDay

AND nEstado = 1

)

BREAK;

  

SET @NextDay = DATEADD(DAY, 1, @NextDay);

END

  

RETURN @NextDay;

END;