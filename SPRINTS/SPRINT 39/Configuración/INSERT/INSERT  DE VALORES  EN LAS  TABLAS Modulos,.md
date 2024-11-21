INSERT INTO Modulos
(nPadre, sDescripcion, sComentario, sIcono, sUrl_Link_Modulo, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, nOrden, nEstado_Config, nId_moduloRol, v_Mostrar)
VALUES(NULL,'Solicitudes', 'Política de vencimiento de Permisos', 'add_notes', NULL, 1, 756, getdate(), NULL, NULL, NULL, NULL, 31, 1, NULL, 0)


ID_Modulo: 32

INSERT INTO opciones
(nFuncionalidad_Tipo, sDescripcion, sComentario, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, sSlug, nId_Modulo)
VALUES(1, 'CREAR SOLICITUDES', 'Acceso para crear un solicitudes', 1, 756, getdate(), NULL, NULL, NULL, NULL, 'SOLICITUD-CREAR', 32)


nId_Opcion= 277

INSERT INTO permisos_opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(4, 277, NULL, NULL, 1, 756, getdate(), NULL, NULL, NULL, NULL);

INSERT INTO permisos_opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(6, 277, NULL, NULL, 1, 756, getdate(), NULL, NULL, NULL, NULL);


INSERT INTO permisos_opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(9, 277, NULL, NULL, 1, 756, getdate(), NULL, NULL, NULL, NULL);

















