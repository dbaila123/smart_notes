```sql 
IF NOT EXISTS (
	SELECT * FROM Modulos WHERE sDescripcion = 'Evaluaciones' AND v_Mostrar = 1
)
BEGIN
	declare @nid_rrh int, @nid_administracion int, @nid_la int, @nid_lider int, @nid_colaborador int, @nid_director int,  @nid_comunicaciones int, @nid_servicedesk int,  @nid_coordinador int  

	select @nid_rrh = nid_rol from Roles where sDescripcion = 'RRHH'
	select @nid_administracion = nid_rol from Roles where sDescripcion = 'ADMINISTRACIÓN'
	select @nid_la = nid_rol from Roles where sDescripcion = 'LÍDER ADMINISTRATIVO'
	select @nid_lider = nid_rol from Roles where sDescripcion = 'LÍDER'
	select @nid_colaborador = nid_rol from Roles where sDescripcion = 'COLABORADOR'
    select @nid_director = nid_rol from Roles where sDescripcion = 'DIRECTOR'
    select @nid_comunicaciones = nid_rol from Roles where sDescripcion = 'COMUNICACIONES'
    select @nid_servicedesk = nid_rol from Roles where sDescripcion = 'SERVICE DESK'
	select @nid_coordinador = nid_rol from Roles where sDescripcion = 'COORDINADOR'




	declare @nid_modulo int
	INSERT INTO Modulos
	(nPadre, sDescripcion, sComentario, sIcono, sUrl_Link_Modulo, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, nOrden, nEstado_Config, nId_moduloRol, v_Mostrar)
	VALUES(NULL,'Evaluaciones', 'Evaluación de Desempeño', 'add_notes', '/evaluacion/lista', 1, 172, getdate(), NULL, NULL, NULL, NULL, null, 1, NULL, 0)
	set @nid_modulo = SCOPE_IDENTITY() 

	update Modulos set nOrden = @nid_modulo,  nId_moduloRol = @nid_modulo where nId_Modulo = @nid_modulo



	declare @nid_opcion int
	INSERT INTO opciones
	(nFuncionalidad_Tipo, sDescripcion, sComentario, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, sSlug, nId_Modulo)
	VALUES(1, 'MODULO DE EVALUACIÓN DE DESEMPEÑO', 'Modulo de Evaluación de Desempeño', 1, 172, getdate(), NULL, NULL, NULL, NULL, 'EVALUACION-DESEMP-LISTA', @nid_modulo)
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



	INSERT INTO permisos_opciones
	(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
	VALUES(@nid_lider, @nid_opcion, NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);


	INSERT INTO permisos_opciones
	(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
	VALUES(@nid_colaborador, @nid_opcion, NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

	INSERT INTO permisos_opciones
	(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
	VALUES(@nid_director, @nid_opcion, NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

	
	INSERT INTO permisos_opciones
	(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
	VALUES(@nid_comunicaciones, @nid_opcion, NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

	INSERT INTO permisos_opciones
	(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
	VALUES(@nid_servicedesk, @nid_opcion, NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);
	
	INSERT INTO permisos_opciones
	(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
	VALUES(@nid_coordinador, @nid_opcion, NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);

END


```