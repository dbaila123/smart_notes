```SQL
CREATE FUNCTION fg_get_calcular_estado_marca(

@id_colaborador INT,

@fecha_asistencia DATE

)

RETURNS VARCHAR(2)

AS

BEGIN

DECLARE @estado VARCHAR(2);

DECLARE @num_marcas INT;

DECLARE @num_extemporaneas INT;

DECLARE @num_no_marcas INT;

  

-- Contar el número de marcas (si se realizaron marcas)

SELECT @num_marcas = COUNT(*)

FROM marcas

WHERE nId_Colaborador = @id_colaborador

AND CAST(dFecha_Marca AS DATE) = @fecha_asistencia

AND nEstado=1 ;

  

-- Contar el número de marcas extemporáneas

SELECT @num_extemporaneas = COUNT(*)

FROM marcas

WHERE nId_Colaborador = @id_colaborador

AND CAST(dFecha_Marca AS DATE) = @fecha_asistencia

AND nEstado = 0;

  

-- Contar el número de marcas que nunca se hicieron (estado = 2)

SELECT @num_no_marcas = COUNT(*)

FROM marcas

WHERE nId_Colaborador = @id_colaborador

AND CAST(dFecha_Marca AS DATE) = @fecha_asistencia

  

-- Determinar el estado basado en las reglas

IF @num_marcas = 0

BEGIN

SET @estado = 'NL'; -- No laboró

END

ELSE

BEGIN

IF @num_no_marcas > 0

BEGIN

SET @estado = 'NL'; -- No laboró (porque hay marcas con estado 2)

END

ELSE

BEGIN

IF @num_extemporaneas > 0

BEGIN

SET @estado = 'CE'; -- Con extemporánea

END

ELSE

BEGIN

SET @estado = 'SE'; -- Sin extemporánea

END

END

END

  

-- Retornar el estado calculado

RETURN @estado;

END;
```