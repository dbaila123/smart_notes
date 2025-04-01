```sql
CREATE OR ALTER  FUNCTION fn_total_min_days_by_schedule_Collaborator_V2 (
   @dFecha_Inicio DATE,
   @dFecha_Fin DATE,
   @nId_Colaborador INT
)
RETURNS INT
AS
BEGIN

    DECLARE @nCantidad_Minutos INT;
    DECLARE @nDias TABLE (nDia INT);
   	DECLARE @nId_Horario INT;

    SELECT @nId_Horario = nId_Horario 
    FROM Horarios_Colaboradores hc 
    WHERE nId_Colaborador = @nId_Colaborador AND nEstado = 1;
   
    WITH RangoFechas AS 
(
    SELECT @dFecha_Inicio AS Fecha
    UNION ALL
    SELECT DATEADD(DAY, 1, Fecha)
    FROM RangoFechas
    WHERE Fecha < @dFecha_Fin
    )
    INSERT INTO @nDias  (nDia)
    SELECT
        dbo.fn_get_weekday(Fecha) AS DiaSemana
    FROM
        RangoFechas;

    -- Calcular los minutos acumulados por rango de días
    SELECT
        @nCantidad_Minutos = SUM(hd.nTotalJornada * 60) -- Convertir horas a minutos
    FROM
        Horarios_Detalles hd
   --JOIN
       -- Asistencias a ON a.nId_Horario = hd.nId_Horario AND dbo.fn_get_weekday(a.dFecha_Asistencia) = hd.nDia
   WHERE hd.nId_Horario = @nId_Horario
      AND hd.nEstado = 1
      AND hd.nDia IN (SELECT nDia FROM @nDias);


    RETURN @nCantidad_Minutos;
END

```
