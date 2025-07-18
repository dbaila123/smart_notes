```SQL

-- Opciones
INSERT INTO Opciones (nFuncionalidad_Tipo, sDescripcion, sComentario, nEstado, nUsuario_Creador, dDatetime_Creador, sSlug, nId_Modulo)
VALUES
(2, 'DASHBOARD INDICADORES COLABORADORES', 'Dashboard de Horas favor/contra - Colaboradores', 1, 172, GETDATE(), 'DASHBOARD-HOURS-INDICATORS', 32);

INSERT INTO Opciones (nFuncionalidad_Tipo, sDescripcion, sComentario, nEstado, nUsuario_Creador, dDatetime_Creador, sSlug, nId_Modulo)
VALUES
(2, 'DASHBOARD INDICADORES COLABORADORES - EQUIPOS', 'Dashboard de Horas favor/contra - Colaboradores', 1, 172, GETDATE(), 'DASHBOARD-HOURS-INDICATORS-TEAM', 32);

-- Permisos_Opciones
INSERT INTO Permisos_Opciones  (nId_Rol, nId_Opcion, nEstado, nUsuario_Creador, dDatetime_Creador)
VALUES
(1, 281, 1, 172, GETDATE()),
(4, 281, 1, 172, GETDATE()),
(6, 281, 1, 172, GETDATE()),
(9, 281, 1, 172, GETDATE()),
(17, 281, 1, 172, GETDATE());

INSERT INTO Permisos_Opciones  (nId_Rol, nId_Opcion, nEstado, nUsuario_Creador, dDatetime_Creador)
VALUES
(5, 282, 1, 172, GETDATE()),
(16, 282, 1, 172, GETDATE());

```