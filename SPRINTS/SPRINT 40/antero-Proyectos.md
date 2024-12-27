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
UPDATE Requerimientos

SET sNombre_SinPrefijo = LTRIM(RTRIM(SUBSTRING(sNombre, 5, LEN(sNombre))))

WHERE sNombre LIKE 'RQ - %';
```