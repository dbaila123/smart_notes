```SQL

-- Actualizar nombre de proyectos con prefijo o código
UPDATE Proyectos

SET sNombre = CASE

WHEN sCodigo = 'PRY - ' THEN 'PRY - ' + sNombre

WHEN sCodigo = 'MAN - ' THEN 'MAN - ' + sNombre

ELSE sCodigo + ' - ' + sNombre

END

WHERE sCodigo IS NOT NULL

-- Actualizar código de proyectos si tienen el prefijo como código  

UPDATE Proyectos

SET sCodigo = '-'

WHERE sCodigo in ('PRY - ', 'MAN - ')

-- Actualizar nombre de requerimientos con prefijo o código

UPDATE Requerimientos

SET sNombre = CASE

WHEN sCodigo = 'RQ - ' THEN 'RQ - ' + sNombre

ELSE sCodigo + ' - ' + sNombre

END

WHERE sCodigo IS NOT NULL

-- Actualizar codigo de requerimientos si tienen el prefijo como código

UPDATE Requerimientos

SET sCodigo = '-'

WHERE sCodigo = 'RQ - '