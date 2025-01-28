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

  

-- Contar las marcas existentes

SELECT @marca_existente = COUNT(*)

FROM Marcas

WHERE nEstado = 1

AND nId_Colaborador = @id_colaborador

AND dFecha_Jornada = @fecha_asistencia;

  

IF @marca_existente > 0

BEGIN

---contar marcas extemporaneas

SELECT @marca_extemporaneas = COUNT(*)

FROM Marcas

WHERE nMarcaje_Extemporaneo = 1

AND nEstado = 1

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

-- No hay existe marca

SET @estado = '0';

END

  

RETURN @estado;

END;
```