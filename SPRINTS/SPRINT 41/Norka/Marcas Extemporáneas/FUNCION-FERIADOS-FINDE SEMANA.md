CREATE FUNCTION fg_get_EsFeriadoOFinDeSemana
(
    @fecha_consulta DATE
)
RETURNS INT
AS
BEGIN
    DECLARE @es_feriado INT;
    DECLARE @dia_semana INT;

    -- Verificar si la fecha es un feriado
    SELECT @es_feriado = COUNT(*)
    FROM Feriados
    WHERE dFecha = @fecha_consulta;

    -- Verificar si la fecha es un fin de semana (sábado o domingo)
    SET @dia_semana = DATEPART(WEEKDAY, @fecha_consulta);

    -- Determinar si es feriado o fin de semana
    IF @es_feriado > 0 OR @dia_semana = 1 OR @dia_semana = 7
        RETURN 1; -- Es feriado o fin de semana
    ELSE
        RETURN 0; -- No es feriado ni fin de semana

    RETURN 0; -- Por defecto, retorna 0 (no es feriado ni fin de semana)
END;