```SQL
CREATE FUNCTION fg_get_calcular_estado_marca_extemp(

@id_colaborador INT,

@fecha_asistencia DATE

)

RETURNS VARCHAR(2)

AS

BEGIN

DECLARE @estado VARCHAR(2);

DECLARE @marca_existente INT;

DECLARE @marca_extemporaneas INT;

DECLARE @num_no_marcas INT;

  

-- Contar las marcas existentes, solo de colaboradores activos

SELECT @marca_existente = count(*)

FROM Asistencias a

INNER JOIN Colaboradores c ON a.nId_Colaborador = c.nId_Colaborador

INNER JOIN Horarios_Colaboradores hc on c.nId_Colaborador = hc.nId_Colaborador AND hc.nEstado = 1

INNER JOIN Horarios_Detalles hd on hc.nId_Horario = hd.nId_Horario and

CASE

WHEN hd.nDia = 0 THEN 'sunday'

WHEN hd.nDia = 1 THEN 'monday'

WHEN hd.nDia = 2 THEN 'tuesday'

WHEN hd.nDia = 3 THEN 'wednesday'

WHEN hd.nDia = 4 THEN 'thursday'

WHEN hd.nDia = 5 THEN 'friday'

WHEN hd.nDia = 6 THEN 'saturday'

END = LOWER(DATENAME(WEEKDAY, a.dFecha_Asistencia))

WHERE a.nTiene_Marca > 0

AND a.nId_Colaborador = @id_colaborador

AND a.dFecha_Asistencia = @fecha_asistencia

--AND c.nEstado_Colaborador=1;

  

IF @marca_existente > 0

BEGIN

---contar marcas extemporaneas en estado aprobrado

SELECT @marca_extemporaneas = COUNT(*)

FROM Marcas

WHERE nMarcaje_Extemporaneo = 1

AND nEstado IN (0,1)

AND nId_Colaborador = @id_colaborador

AND dFecha_Jornada = @fecha_asistencia;

  

IF @marca_extemporaneas > 0

BEGIN

-- Día con extemporánea

SET @estado = '2';

END

ELSE

BEGIN

-- Día normal

SET @estado = '1';

END

END

ELSE

BEGIN

-- No existe marca

SET @estado = '0';

END

  

RETURN @estado;

END;