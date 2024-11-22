IF NOT EXISTS (
	SELECT * FROM Modulos WHERE sDescripcion = 'Solicitudes' AND v_Mostrar = 0
)
BEGIN
	declare @nid_rrh int, @nid_administracion int, @nid_la int
	select @nid_rrh = nid_rol from Roles where sDescripcion = 'RRHH'
	select @nid_administracion = nid_rol from Roles where sDescripcion = 'ADMINISTRACIÓN'
	select @nid_la = nid_rol from Roles where sDescripcion = 'LÍDER ADMINISTRATIVO'


	declare @nid_modulo int
	INSERT INTO Modulos
	(nPadre, sDescripcion, sComentario, sIcono, sUrl_Link_Modulo, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, nOrden, nEstado_Config, nId_moduloRol, v_Mostrar)
	VALUES(NULL,'Solicitudes', 'Política de vencimiento de Permisos', 'add_notes', NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL, null, 1, NULL, 0)
	set @nid_modulo = SCOPE_IDENTITY() 

	update Modulos set nOrden = @nid_modulo where nId_Modulo = @nid_modulo


	declare @nid_opcion int
	INSERT INTO opciones
	(nFuncionalidad_Tipo, sDescripcion, sComentario, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, sSlug, nId_Modulo)
	VALUES(1, 'SOLICITUDES DE VENCIMIENTO', 'Configuración de Vencimiento de Solicitudes', 1, 172, getdate(), NULL, NULL, NULL, NULL, 'SOLICITUD-VENCIMIENTO-UPDATE', @nid_modulo)
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
GO
