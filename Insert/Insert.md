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


ALTER TABLE Transacciones_Saldo_Mins
ADD CONSTRAINT FK_Transacciones_Closure_Request
FOREIGN KEY (nId_Cierre) REFERENCES closure_Request(nId_Cierre);


create table closure_Request
    (nId_Cierre INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
	nEstado_Cierre int NULL,
	nEstado int NULL,
	dFecha_Cierre date NULL,
	dDatetime_Create datetime NULL,
	dDatetime_Update datetime NULL,
	nUsuario_Creador int NULL,
	nUsuario_Update int NULL,
	dDatetime_Delete datetime NULL,
	nUsuario_Delete int NULL);
	
```