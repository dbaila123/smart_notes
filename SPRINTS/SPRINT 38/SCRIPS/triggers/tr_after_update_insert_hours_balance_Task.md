```sql
CREATE TRIGGER tr_after_update_insert_hours_balance_Task
ON Tareas
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
   DECLARE @nId_Tipo_Entidad INT = 9
	DECLARE @nId_Sub_Tipo_Entidad INT =8
	
    INSERT INTO Transacciones_Saldo_Mins 
    (
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
    dDatetime_Update,
    nUsuario_Update   
    )
    SELECT 
        d.nId_Colaborador,
        d.nEstado,
        i.nEstado_Tarea,
        @nId_Tipo_Entidad,
        @nId_Sub_Tipo_Entidad,
        d.nId_Tarea,
         CASE 
            --WHEN d.nId_Tipo_Solicitud IN (1, 2) THEN 2
            WHEN d.sTipo_Hora IN (2,3,5) THEN 1
            ELSE NULL
        END AS nTipo_Transaccion,
        d.sDetalle,
        d.nMinutos,
        d.nUsuario_Creador,
        d.dDatetime_Creador,
        null,
        d.dDatetime_Update,
        d.nUsuario_Update
    FROM inserted i
    JOIN deleted d ON i.nId_Tarea  = d.nId_Tarea
     WHERE i.nEstado_Tarea = 3
      AND (d.nEstado_Tarea IS NULL OR d.nEstado_Tarea <> 3);
     
         -- Eliminación de transacciones cuando nEstado_Solicitud cambia a 0
    DELETE FROM Transacciones_Saldo_Mins
    WHERE nId_Entidad IN (SELECT d.nId_Tarea 
                          FROM deleted d 
                          JOIN inserted i ON d.nId_Tarea = i.nId_Tarea
                          WHERE i.nEstado_Tarea = 6);
END;


```