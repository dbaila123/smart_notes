```sql
insert into Entidades_Transacciones (nEstado, nTipo_Entidad, nId_Entidad_Padre, sDescripcion, 
dDatetime_Creacion,nUsuario_Creador,dDatetime_Update,nUsuario_Update)
VALUES(1,2,1,'HORAS A FAVOR',
'2024-10-24 15:30:33',172,NULL,NULL)



insert into Entidades_Transacciones (nEstado, nTipo_Entidad, nId_Entidad_Padre, sDescripcion, 
dDatetime_Creacion,nUsuario_Creador,dDatetime_Update,nUsuario_Update)
VALUES(1,1,NULL,'TAREO',
'2024-10-24 15:30:33',172,NULL,NULL)


alter table Tipos_Solicitudes 
add b_recuperable bit



update Tipos_Solicitudes 
set b_recuperable = 1 
where nId_Tipo_Solicitud in (1,2)
```