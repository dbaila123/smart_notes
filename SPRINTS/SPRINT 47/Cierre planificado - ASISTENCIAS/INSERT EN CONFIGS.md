```SQL
INSERT INTO Configs (sTabla, sCodigo, sDescripcion, sComentario, nOrdenamiento, sCodigo_Externo, nEstado, nUsuario_Creador, dDatetime_Creador)

VALUES

('ESTADO_INDICADORES', '1', 'APROBADO', 'Estado aprobado de horas en indicadores', 1, 'EI', 1, 172, GETDATE()),

('ESTADO_INDICADORES', '2', 'PENDIENTE', 'Estado pendiente de horas en indicadores', 2, 'EI', 1, 172, GETDATE()),

('ESTADO_INDICADORES', '3', 'PRE-PLANILLA', 'Estado pre-planilla de horas en indicadores', 3, 'EI', 1, 172, GETDATE());