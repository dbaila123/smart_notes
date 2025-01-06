```sql
alter table Proyectos

add sCodigo varchar(max)
----------------
alter table Requerimientos

add sCodigo varchar(max)
----------------
alter table Requerimientos

add sNombre_SinPrefijo varchar(max)
----------------
alter table Requerimientos

add sCodigo_SinPrefijo varchar(max)
----------------
UPDATE Requerimientos

SET sNombre_SinPrefijo = LTRIM(RTRIM(SUBSTRING(sNombre, 5, LEN(sNombre))))

WHERE sNombre LIKE 'RQ - %';
----------------
UPDATE Requerimientos

SET sNombre_SinPrefijo = LTRIM(RTRIM(SUBSTRING(sNombre, 5, LEN(sNombre))))

WHERE sNombre LIKE 'RQ -%';

--------------ACTUALIZACIÓN REQUERIMIENTOS SCODIGO-----------------

UPDATE Requerimientos set sCodigo = CONCAT_WS('', 'RQ', nId_Requerimiento) where sCodigo = '' or sCodigo is null;

---------------ACTUALIZACIÓN REQUERIMINETOS SNOMBRE----------------

UPDATE Requerimientos set sNombre = CONCAT_WS(' ', sCodigo, sNombre_SinPrefijo)

----------------ACTUALIZAR PROYECTOS SIN PROYECTOS-----------------

UPDATE Proyectos set sNombre_Proyecto_Sin_Prefijo = sDescripcion where sNombre_Proyecto_Sin_Prefijo = '' or sNombre_Proyecto_Sin_Prefijo is null

--------------ACTUALIZACIÓN PROYECTOS SCODIGO----------------------

UPDATE Proyectos set sCodigo = CONCAT_WS('', 'PRY', nId_Proyecto) where sCodigo = '' or sCodigo is null;

---------------ACTUALIZACIÓN PROYECTOS SNOMBRE---------------------

UPDATE Proyectos set sNombre = CONCAT_WS(' ', sCodigo, sNombre_Proyecto_Sin_Prefijo)
```