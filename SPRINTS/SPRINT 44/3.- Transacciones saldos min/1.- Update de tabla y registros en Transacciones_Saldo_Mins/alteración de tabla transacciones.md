```sql
ALTER TABLE Transacciones_Saldo_Mins
ADD nEstado_Entidad INT NULL;

UPDATE Transacciones_Saldo_Mins
SET nEstado_Entidad = 3
WHERE nEstado_Entidad IS NULL;

-- Query de actualización recuperacion y compensacion pendientes a la tabla transacciones
INSERT INTO Transacciones_Saldo_Mins (
    nId_Colaborador,
    nEstado,
    nEstado_Transaccion,
    nId_Tipo_Entidad,
    nId_Sub_Tipo_Entidad,
    nId_Entidad,
    nTipo_Transaccion,
    sObservacion,
    dCantidad_Minutos,
    nUsuario_Creador,
    dDatetime_Creacion,
    nId_Cierre,
	nEstado_Entidad
)
SELECT 
    s.nId_Colaborador,
    1 AS nEstado,
    1 AS nEstado_Transaccion,
    9 AS nId_Tipo_Entidad,
    8 AS nId_Sub_Tipo_Entidad,
    s.nId_Tarea AS nId_Entidad,
    1 AS nTipo_Transaccion,
    NULL AS sObservacion,
    s.nCantidad AS dCantidad_Minutos,
    u.nId_usuario AS nUsuario_Creador,
    s.dDatetime_Creador AS dDatetime_Creacion,
    NULL AS nId_Cierre,
	1 AS nEstado_Entidad
FROM solicitudes s
LEFT JOIN colaboradores c ON c.nId_Colaborador = s.nId_Colaborador
LEFT JOIN personas p ON p.nId_Persona = c.nId_Persona
LEFT JOIN usuarios u ON u.nId_Persona = p.nId_Persona
WHERE s.nId_Tipo_Solicitud IN (7,8) 
  AND s.nEstado_Solicitud IN (1,4)
ORDER BY s.dFecha_Solicitud;

-- Query de actualización permisos pendientes a la tabla transacciones
INSERT INTO Transacciones_Saldo_Mins (
    nId_Colaborador,
    nEstado,
    nEstado_Transaccion,
    nId_Tipo_Entidad,
    nId_Sub_Tipo_Entidad,
    nId_Entidad,
    nTipo_Transaccion,
    sObservacion,
    dCantidad_Minutos,
    nUsuario_Creador,
    dDatetime_Creacion,
    nId_Cierre,
	nEstado_Entidad
)
SELECT 
    s.nId_Colaborador,
    1 AS nEstado,
    1 AS nEstado_Transaccion,
    1 AS nId_Tipo_Entidad,
    6 AS nId_Sub_Tipo_Entidad,
    s.nId_Solictud AS nId_Entidad,
    1 AS nTipo_Transaccion,
    NULL AS sObservacion,
    s.nCantidad AS dCantidad_Minutos,
    u.nId_usuario AS nUsuario_Creador,
    s.dDatetime_Creador AS dDatetime_Creacion,
    NULL AS nId_Cierre,
	1 AS nEstado_Entidad
FROM solicitudes s
LEFT JOIN colaboradores c ON c.nId_Colaborador = s.nId_Colaborador
LEFT JOIN personas p ON p.nId_Persona = c.nId_Persona
LEFT JOIN usuarios u ON u.nId_Persona = p.nId_Persona
WHERE s.nId_Tipo_Solicitud IN (1,2) 
  AND s.nEstado_Solicitud IN (1,4)
ORDER BY s.dFecha_Solicitud;
```
