```sql
alter procedure sp_smart_get_descuentos_solicitudes
(
	@nId INT
)
 AS
 BEGIN
 		SELECT 
 	c.nId_Colaborador,
 	p.sPersona_Nombre AS SNombreColaborador,
 	tsm.nId_Sub_Tipo_Entidad,
 	tsm.nId_Tipo_Entidad,
 	tsm.nTipo_Transaccion,
    s.nId_Tipo_Solicitud,
    s.nId_Solictud,
    s.dDatetime_Creador,
    s.nUsuario_Creador,
    s.nCantidad,
    s.dHora_Inicio,
    s.dHora_Fin,
    s.sMotivo,
    s.nId_Supervisor,
    s.nId_Tipo_Motivo,
    s.sObservacion_Lider
    FROM Transacciones_Saldo_Mins tsm
    JOIN Solicitudes s ON tsm.nId_Entidad = s.nId_Solictud
    JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador 
    JOIN Personas p ON p.nId_Persona = c.nId_Persona 
    join Tipos_Solicitudes ts on ts.nId_Tipo_Solicitud  = s.nId_Tipo_Solicitud 
    WHERE tsm.nId_Colaborador = @nId and 
    ts.b_recuperable = 1
    AND nId_Cierre IS NULL;
END
```