```sql
insert into Entidades_Transacciones (nEstado, nTipo_Entidad, nId_Entidad_Padre, sDescripcion, 
dDatetime_Creacion,nUsuario_Creador,dDatetime_Update,nUsuario_Update)
VALUES(1,2,1,'HORAS A FAVOR',
'2024-10-24 15:30:33',172,NULL,NULL)



  
update Entidades_Transacciones  
set sDescripcion =  'DESCUENTOS'  
where sDescripcion =  'TARDANZAS A DESCUENTO'


alter table Tipos_Solicitudes 
add b_recuperable bit

INSERT INTO Configs (sTabla, sCodigo, sDescripcion, sComentario, sCodigo_Externo, nEstado, nUsuario_Creador, nOrdenamiento, dDatetime_Creador)  
VALUES  
('VENCIMIENTO-SOLICITUDES', 30, 'TIEMPO DE VENCIMIENTO SOLICITUDES','TIEMPO PARA DESCUENTO DE LA SOLICITUD', 'T-V-S', 1, 172, 1, GETDATE())

ALTER TABLE Transacciones_Saldo_Mins  
DROP CONSTRAINT DF__Transacci__sObse__68DD7AB4;  
  
ALTER TABLE Transacciones_Saldo_Mins  
ALTER COLUMN sObservacion NVARCHAR(MAX);  
  
ALTER TABLE Transacciones_Saldo_Mins  
ADD CONSTRAINT DF__Transacci__sObse__68DD7AB4 DEFAULT null FOR sObservacion;


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

	CREATE TABLE History_Closing_Transactions(    
	nId_Cierre INT PRIMARY KEY,    
	dDatetime_Creacion DATETIME,    
	nUsuario_Creador INT,    
	dCantidad_Minutos INT,    
	Json_Detalles_Transacciones NVARCHAR(MAX)
	);
```

```
INSERT INTO Opciones
(nId_Modulo, nFuncionalidad_Tipo, sDescripcion, sComentario, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, sSlug)
VALUES((SELECT nId_Modulo from Modulos where sDescripcion='Colaboradores' AND sUrl_Link_Modulo = '/colaboradores/lista'), 1, 'CARD DE HORAS A FAVOR Y EN CONTRA', 'Card de horas a favor y en contra', 1, 172, '2024-04-10 12:18:00.000', NULL, NULL, NULL, NULL, 'POSITIVE-NEGATIVE-HOURS-INDIVIDUAL');

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(6, (SELECT nId_Opcion from Opciones where sSlug='POSITIVE-NEGATIVE-HOURS-INDIVIDUAL'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(4, (SELECT nId_Opcion from Opciones where sSlug='POSITIVE-NEGATIVE-HOURS-INDIVIDUAL'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(9, (SELECT nId_Opcion from Opciones where sSlug='POSITIVE-NEGATIVE-HOURS-INDIVIDUAL'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(3, (SELECT nId_Opcion from Opciones where sSlug='POSITIVE-NEGATIVE-HOURS-INDIVIDUAL'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(5, (SELECT nId_Opcion from Opciones where sSlug='POSITIVE-NEGATIVE-HOURS-INDIVIDUAL'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);
```

```sql
update pc
	set pc.sCards_Orden = REPLACE(pc.sCards_Orden, ']]', ',12]]')
from PreferenciasColaborador pc
inner join Colaboradores col on col.nid_colaborador = pc.nid_colaborador
inner join Personas p on p.nId_Persona = col.nId_Colaborador
inner join Usuarios u on u.nId_Persona = p.nId_Persona
inner join Roles_Usuarios ru on ru.nId_Usuario = u.nId_Usuario
inner join Permisos_Opciones pos on pos.nId_Rol = ru.nId_Rol
inner join opciones op on op.nId_Opcion = pos.nId_Opcion
where op.sSlug = 'POSITIVE-NEGATIVE-HOURS-INDIVIDUAL'
```