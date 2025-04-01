```sql
------------------------COLABORADORES--------------------------------
insert into Opciones 
(
	nFuncionalidad_Tipo,sDescripcion,sComentario,nEstado,nUsuario_Creador,dDatetime_Creador,sSlug,nId_Modulo
) 
values 
(
	2, 'INDICADORES-COLABORADORES', 'Filtro de colaboradores', 1, 172, GETDATE(), 'INDICADORES-MODULO-COLABORADORES', (select nId_Modulo from Modulos where sDescripcion like '%Indicadores%')
);

DECLARE @nId_Opciones INT = SCOPE_IDENTITY()


INSERT INTO Permisos_Opciones (nId_Rol, nId_Opcion, nEstado, nUsuario_Creador, dDatetime_Creador)  
VALUES   
       (4, @nId_Opciones, 1, 172, GETDATE()),  
       (5, @nId_Opciones, 0, 172, GETDATE()),  
       (6, @nId_Opciones, 1, 172, GETDATE()),  
       (9, @nId_Opciones, 1, 172, GETDATE());
------------------------SOLICITUDES--------------------------------
insert into Opciones 
(
	nFuncionalidad_Tipo,sDescripcion,sComentario,nEstado,nUsuario_Creador,dDatetime_Creador,sSlug,nId_Modulo
) 
values 
(
	2, 'INDICADORES-SOLICITUDES', 'Filtros de solicitudes', 1, 172, GETDATE(), 'INDICADORES-MODULO-SOLICITUDES', (select nId_Modulo from Modulos where sDescripcion like '%Indicadores%')
);

DECLARE @nId_Opciones INT = SCOPE_IDENTITY()


INSERT INTO Permisos_Opciones (nId_Rol, nId_Opcion, nEstado, nUsuario_Creador, dDatetime_Creador)  
VALUES   
       (4, @nId_Opciones, 1, 172, GETDATE()),  
       (5, @nId_Opciones, 1, 172, GETDATE()),  
       (6, @nId_Opciones, 1, 172, GETDATE()),  
       (9, @nId_Opciones, 1, 172, GETDATE());
--------------------TAREO-------------------------------
insert into Opciones 
(
	nFuncionalidad_Tipo,sDescripcion,sComentario,nEstado,nUsuario_Creador,dDatetime_Creador,sSlug,nId_Modulo
) 
values 
(
	2, 'INDICADORES-TAREO', 'Filtros de solicitudes', 1, 172, GETDATE(), 'INDICADORES-MODULO-TAREO', (select nId_Modulo from Modulos where sDescripcion like '%Indicadores%')
);

DECLARE @nId_Opciones INT = SCOPE_IDENTITY()


INSERT INTO Permisos_Opciones (nId_Rol, nId_Opcion, nEstado, nUsuario_Creador, dDatetime_Creador)  
VALUES   
       (4, @nId_Opciones, 1, 172, GETDATE()),  
       (5, @nId_Opciones, 1, 172, GETDATE()),  
       (6, @nId_Opciones, 1, 172, GETDATE()),  
       (9, @nId_Opciones, 1, 172, GETDATE()),
       (10, @nId_Opciones, 1, 172, GETDATE());
----------------------PROYECTO-----------------------
insert into Opciones 
(
	nFuncionalidad_Tipo,sDescripcion,sComentario,nEstado,nUsuario_Creador,dDatetime_Creador,sSlug,nId_Modulo
) 
values 
(
	2, 'INDICADORES-PROYECTO', 'Filtros de solicitudes', 1, 172, GETDATE(), 'INDICADORES-MODULO-PROYECTO', (select nId_Modulo from Modulos where sDescripcion like '%Indicadores%')
);

DECLARE @nId_Opciones INT = SCOPE_IDENTITY()


INSERT INTO Permisos_Opciones (nId_Rol, nId_Opcion, nEstado, nUsuario_Creador, dDatetime_Creador)  
VALUES   
       (4, @nId_Opciones, 1, 172, GETDATE()),  
       (5, @nId_Opciones, 1, 172, GETDATE()),  
       (6, @nId_Opciones, 1, 172, GETDATE()),  
       (9, @nId_Opciones, 1, 172, GETDATE()),
       (10, @nId_Opciones, 1, 172, GETDATE());
---------------------------LIDERES--------------------------
insert into Opciones 
(
	nFuncionalidad_Tipo,sDescripcion,sComentario,nEstado,nUsuario_Creador,dDatetime_Creador,sSlug,nId_Modulo
) 
values 
(
	2, 'INDICADORES-COLABORADORES-EQUIPOS', 'Filtros de equipos en colaboradores', 1, 172, GETDATE(), 'INDICADORES-MODULO-COLABORADORES-EQUIPOS', (select nId_Modulo from Modulos where sDescripcion like '%Indicadores%')
);

DECLARE @nId_Opciones INT = SCOPE_IDENTITY()


INSERT INTO Permisos_Opciones (nId_Rol, nId_Opcion, nEstado, nUsuario_Creador, dDatetime_Creador)  
VALUES     
       (5, @nId_Opciones, 1, 172, GETDATE()),  
```