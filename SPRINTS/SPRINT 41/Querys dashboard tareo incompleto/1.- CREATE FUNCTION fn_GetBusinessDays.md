```SQL
CREATE FUNCTION fn_GetBusinessDays

(

@StartDate DATE,

@EndDate DATE

)

RETURNS INT

AS

BEGIN

DECLARE @BusinessDays INT = 0

DECLARE @CurrentDate DATE = @StartDate

  

WHILE @CurrentDate <= @EndDate

BEGIN

-- Verificar si no es fin de semana (1 = Domingo, 7 = Sábado)

-- Verificar que no sea feriado

IF DATEPART(WEEKDAY, @CurrentDate) NOT IN (1, 7)

AND NOT EXISTS (

SELECT 1

FROM Feriados

WHERE dFecha = @CurrentDate

AND nEstado = 1

)

BEGIN

SET @BusinessDays = @BusinessDays + 1

END

SET @CurrentDate = DATEADD(DAY, 1, @CurrentDate)

END

  

RETURN @BusinessDays

END;