```sql
CREATE OR ALTER  TRIGGER tr_after_Transacciones_Saldo_Mins    
ON Solicitudes    
AFTER UPDATE, INSERT    
AS    
BEGIN    
    SET NOCOUNT ON;    
    -- Eliminación de transacciones cuando nEstado_Solicitud YA NO ES APROBADO    
DELETE FROM Transacciones_Saldo_Mins    
WHERE EXISTS (    
    SELECT 1    
    FROM INSERTED i    
    WHERE Transacciones_Saldo_Mins.nId_Entidad =     
            CASE     
                WHEN i.nId_Tipo_Solicitud IN(7,8) THEN i.nId_Tarea    
                ELSE i.nId_Solictud    
            END    
        AND Transacciones_Saldo_Mins.nId_Tipo_Entidad =     
            CASE     
                WHEN i.nId_Tipo_Solicitud IN(7,8) THEN 9    
                ELSE 1    
            END    
);    
    
    
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
        CASE    
            WHEN i.nId_Tipo_Solicitud IN(7,8) THEN 9    
            ELSE 1    
  END AS nId_Tipo_Entidad,    
        CASE    
            WHEN i.nId_Tipo_Solicitud IN (7,8) THEN 8    
            WHEN i.nId_Tipo_Solicitud = 26 THEN (    
                SELECT nId_Entidad FROM Entidades_Transacciones WHERE sDescripcion = 'SOLICITUDES VENCIDAS'    
            )    
            ELSE (    
                SELECT dbo.fn_nEntidad_Transaction_By_Request_Date(i.dFecha_Inicio, CONVERT(DATE, GETDATE()))    
            )    
        END AS nId_Sub_Tipo_Entidad,    
        CASE     
   WHEN i.nId_Tipo_Solicitud IN(7,8) THEN i.nId_Tarea    
   ELSE i.nId_Solictud    
  END AS nId_Entidad,    
        1 AS nTipo_Transaccion,    
        i.sMotivo,    
  CASE    
            WHEN i.nId_Tipo_Solicitud in(1,26,7,8) THEN i.nCantidad  -- Usar el valor de dCantidad_Minutos    
            WHEN i.nId_Tipo_Solicitud = 2 THEN (SELECT  dbo.fn_total_min_days_by_schedule_Collaborator_V2(i.dFecha_Inicio, i.dFecha_Fin, i.nId_Colaborador))    
    
            ELSE NULL    
        END AS dCantidad_Minutos,    
        i.nUsuario_Creador,    
        i.dFecha_Inicio,    
        NULL,    
        i.dDatetime_Update,    
        i.nUsuario_Update,  
  CASE   
   WHEN i.nId_Tipo_Solicitud IN(7,8) THEN i.nEstado_Solicitud  
   ELSE NULL  
  END AS nEstado_Entidad  
    FROM inserted i    
    WHERE (    
 --Acepta solicitudes con tipo = 8 (recuperacion) que sean aprobadas y pendientes    
            (i.nId_Tipo_Solicitud IN (1, 2, 7, 8) AND i.nEstado_Solicitud IN(1,3))  
   OR  
   (i.nId_Tipo_Solicitud = 26 AND i.nEstado_Solicitud = 3))    
 AND NOT EXISTS (    
        SELECT 1    
        FROM Transacciones_Saldo_Mins ts    
        WHERE ts.nId_Entidad =     
                CASE     
                    WHEN i.nId_Tipo_Solicitud IN(7,8) THEN i.nId_Tarea    
                    ELSE i.nId_Solictud    
                END    
            AND ts.nId_Tipo_Entidad =     
                CASE     
                    WHEN i.nId_Tipo_Solicitud IN(7,8) THEN 9    
                    ELSE 1    
                END    
    );    
END;   
```