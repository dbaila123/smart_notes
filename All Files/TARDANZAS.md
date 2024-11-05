- [ ] PERMISOS LISTADO REVISIÓN

```
INSERT INTO Opciones
(nId_Modulo, nFuncionalidad_Tipo, sDescripcion, sComentario, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, sSlug)
VALUES((SELECT nId_Modulo from Modulos where sDescripcion='Asistencia'), 1, 'GESTIÓN DE TARDANZAS', 'Listado de gestión de tardanzas', 1, 172, '2024-04-10 12:18:00.000', NULL, NULL, NULL, NULL, 'ASSISTENCE-TARTINESS');
GO

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(6, (SELECT nId_Opcion from Opciones where sSlug='ASSISTENCE-TARTINESS'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);
GO

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(4, (SELECT nId_Opcion from Opciones where sSlug='ASSISTENCE-TARTINESS'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);
GO

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(9, (SELECT nId_Opcion from Opciones where sSlug='ASSISTENCE-TARTINESS'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);
GO

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(10, (SELECT nId_Opcion from Opciones where sSlug='ASSISTENCE-TARTINESS'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);
GO

  

  

INSERT INTO Opciones
(nId_Modulo, nFuncionalidad_Tipo, sDescripcion, sComentario, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, sSlug)
VALUES((SELECT nId_Modulo from Modulos where sDescripcion='Asistencia'), 1, 'REVISIÓN DE TARDANZAS', 'Realizar revisión de tardanzas', 1, 172, GETDATE(), NULL, NULL, NULL, NULL, 'ASSISTENCE-TARTINESS-REVIEW');
GO

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(6, (SELECT nId_Opcion from Opciones where sSlug='ASSISTENCE-TARTINESS-REVIEW'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);
GO

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(4, (SELECT nId_Opcion from Opciones where sSlug='ASSISTENCE-TARTINESS-REVIEW'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);
GO

  

INSERT INTO Permisos_Opciones
(nId_Rol, nId_Opcion, sUrl_Link_Formulario, sIcono, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
VALUES(9, (SELECT nId_Opcion from Opciones where sSlug='ASSISTENCE-TARTINESS-REVIEW'), NULL, NULL, 1, 172, getdate(), NULL, NULL, NULL, NULL);
GO
```