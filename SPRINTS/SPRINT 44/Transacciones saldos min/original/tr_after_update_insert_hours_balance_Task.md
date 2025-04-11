```sql
CREATE TRIGGER tr_after_update_insert_hours_balance_Task  
ON Tareas  
AFTER UPDATE  
AS  
BEGIN  
    SET NOCOUNT ON;  
   DECLARE @nId_Tipo_Entidad INT = 9  
 DECLARE @nId_Sub_Tipo_Entidad INT =8  
  
             -- Eliminación de transacciones cuando nEstado_Solicitud cambia a 0  
    DELETE FROM Transacciones_Saldo_Mins  
    WHERE nId_Entidad IN (SELECT i.nId_Tarea  
                          FROM inserted i  
                          WHERE i.nEstado_Tarea != 3);  
  
  
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
        i.nUsuario_Update  
    FROM inserted i  
    WHERE i.nEstado_Tarea = 3  
    AND i.sTipo_Hora IN (3,5)  
  
  
END;
```