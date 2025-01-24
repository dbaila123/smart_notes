```SQL
IF NOT EXISTS (

SELECT * FROM opciones WHERE sDescripcion = 'INDICADORES DEL COLABORADOR' AND nId_Modulo = 2

)

BEGIN
declare @nid_rrh int, @nid_administracion int, @nid_la int

select @nid_rrh = nid_rol from Roles where sDescripcion = 'RRHH'
select @nid_administracion = nid_rol from Roles where sDescripcion = 'ADMINISTRACIÓN'
select @nid_la = nid_rol from Roles where sDescripcion = 'LÍDER ADMINISTRATIVO'
declare @nid_opcion int

  
INSERT INTO opciones
(nFuncionalidad_Tipo, sDescripcion, sComentario, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, sSlug, nId_Modulo)
VALUES(2, 'INDICADORES DEL COLABORADOR', 'Indicador de Marcas Extemp. y Tareo Icompleto', 1, 172, getdate(), NULL, NULL, NULL, NULL, 'COLABORADOR-INDICADORES-TODO',2)

set @nid_opcion = SCOPE_IDENTITY()


INSERT INTO permisos_opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)

VALUES(@nid_rrh, @nid_opcion, NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

  
INSERT INTO permisos_opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)

VALUES(@nid_administracion, @nid_opcion, NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);


INSERT INTO permisos_opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)

VALUES(@nid_la, @nid_opcion, NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

END
```
