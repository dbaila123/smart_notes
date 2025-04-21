```sql
CREATE OR ALTER TRIGGER tr_after_update_insert_hours_balance_Task
ON Tareas
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
	DECLARE @IdColaborador NVARCHAR(MAX);
	DECLARE @Date DATE = GETDATE();
	SELECT @IdColaborador = nId_Colaborador
	FROM INSERTED;
    DECLARE @nId_Tipo_Entidad INT = 9
    DECLARE @nId_Sub_Tipo_Entidad INT = 8
    
    -- Eliminar registros para tareas que cambian a estados no deseados
    -- (ni PENDIENTE ni APROBADO para RECUPERACIÓN, o ni APROBADO para COMPENSACIÓN)
    DELETE FROM Transacciones_Saldo_Mins
    WHERE nId_Entidad IN (
       SELECT i.nId_Tarea
        FROM inserted i
        INNER JOIN deleted d ON i.nId_Tarea = d.nId_Tarea
        WHERE (i.nEstado_Tarea NOT IN (1, 3, 9) OR 
              (i.nEstado_Tarea = 1 AND i.sTipo_Hora != 5))
    ) AND nId_Tipo_Entidad = 9;

    -- Actualizar registros existentes cuando cambia el estado
    UPDATE t
    SET t.nEstado 
= i.nEstado,
        t.nEstado_Entidad = CASE WHEN i.nEstado_Tarea = 9 THEN 1 ELSE i.nEstado_Tarea END,
        t.sObservacion = i.sDetalle,
        t.dCantidad_Minutos = i.nMinutos,
        t.dDatetime_Update = i.dDatetime_Update,
        t.nUsuario_Update = i.nUsuario_Update
    FROM Transacciones_Saldo_Mins t
    INNER JOIN inserted i ON t.nId_Entidad = i.nId_Tarea AND t.nId_Tipo_Entidad = 9
    INNER JOIN deleted d ON i.nId_Tarea = d.nId_Tarea
    WHERE (i.nEstado_Tarea != d.nEstado_Tarea OR i.nMinutos != d.nMinutos)
    AND ((i.nEstado_Tarea IN (3, 9) AND i.sTipo_Hora IN (3, 5)) OR 
         (i.nEstado_Tarea IN (1, 9) AND i.sTipo_Hora = 5));

    -- Insertar nuevos registros para tareas recién actualizadas que no tienen registro
    -- (solo si no existe un registro previo)
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
        nUsuario_Update,
        nEstado_Entidad
    )
    SELECT
        i.nId_Colaborador,
        i.nEstado,
  
      1,
        9 AS nId_Tipo_Entidad,
        8 AS nId_Sub_Tipo_Entidad,
        i.nId_Tarea,
        1 AS nTipo_Transaccion,
        i.sDetalle,
        i.nMinutos AS dCantidad_Minutos,
        i.nUsuario_Creador,
        i.dFecha_Registro,
        NULL,
        i.dDatetime_Update,
        i.nUsuario_Update,
        CASE WHEN i.nEstado_Tarea = 9 THEN 1 ELSE i.nEstado_Tarea END AS nEstado_Entidad
    FROM inserted i
    INNER JOIN deleted d ON i.nId_Tarea = d.nId_Tarea
    LEFT JOIN Transacciones_Saldo_Mins t ON 
        i.nId_Tarea = t.nId_Entidad AND t.nId_Tipo_Entidad = 9
    WHERE t.nId_Transaccion IS NULL -- Solo insertar si no existe
    AND (
        (i.nEstado_Tarea IN (3, 9) AND i.sTipo_Hora IN (3, 5)) OR -- APROBADO o PENDIENTE POR DIREC con COMPENSACIÓN o RECUPERACIÓN
        (i.nEstado_Tarea IN (1, 9) AND i.sTipo_Hora = 5) -- PENDIENTE o PENDIENTE POR DIREC solo con RECUPERACIÓN
    );

	exec sp_get_positive_negative_hours_by_Collaborator @dFecha = @Date, @sIds_Colaborador = @IdColaborador
END;
```
