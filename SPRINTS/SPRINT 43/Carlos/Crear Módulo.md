```sql
insert into Modulos 
(
nPadre,	sDescripcion,	sComentario,	sIcono,	sUrl_Link_Modulo,	nEstado,	nUsuario_Creador,	dDatetime_Creador,	nOrden,	nEstado_Config,	nId_moduloRol,	v_Mostrar
) 
values 
(
null, 'Indicadores', 'Módulo para visualizar indicadores', 'finance', '/indicadores/lista', 1, 172, GETDATE(), 18, null, 29, 1
);


insert into Opciones 
(
	nFuncionalidad_Tipo,sDescripcion,sComentario,nEstado,nUsuario_Creador,dDatetime_Creador,sSlug,nId_Modulo
) 
values 
(
	2, 'MODULO INDICADORES', 'Modulo indicadores', 1, 172, GETDATE(), 'INDICADORES-MODULO', (select nId_Modulo from Modulos where sDescripcion like '%Indicadores%')
);

DECLARE @nId_Opciones INT = SCOPE_IDENTITY()


INSERT INTO Permisos_Opciones (nId_Rol, nId_Opcion, nEstado, nUsuario_Creador, dDatetime_Creador)  
VALUES (3,@nId_Opciones, 0, 172, GETDATE()),  
       (4, @nId_Opciones, 0, 172, GETDATE()),  
       (5, @nId_Opciones, 0, 172, GETDATE()),  
       (6, @nId_Opciones, 0, 172, GETDATE()),  
       (9, @nId_Opciones, 0, 172, GETDATE()),  
       (16, @nId_Opciones, 0, 172, GETDATE());


```
