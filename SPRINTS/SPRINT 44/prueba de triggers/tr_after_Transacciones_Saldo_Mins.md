```SQL
CREATE TRIGGER tr_after_Transacciones_Saldo_Mins    
ON Solicitudes    
AFTER UPDATE, INSERT    
AS    
BEGIN    
    DECLARE @IdColaborador NVARCHAR(MAX);
    DECLARE @Date DATE = GETDATE();
    DECLARE @ProcessedByTrigger BIT = 1; -- Marcador para indicar que esta transacción ya fue procesada por el trigger
    
    SELECT @IdColaborador = nId_Colaborador
    FROM INSERTED;
    
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

    -- Actualización de transacciones existentes
    -- Actualizamos y añadimos un comentario indicando que fue procesado por este trigger
    UPDATE Transacciones_Saldo_Mins 
    SET 
        nEstado_Transaccion = 2,
        sObservacion = ISNULL(sObservacion, '') + ' [Procesado por trigger Solicitudes]' -- Marca que identifica quién procesó esto
    FROM Transacciones_Saldo_Mins t
        INNER JOIN Asistencias a ON a.nId_Asistencia = t.nId_Entidad
        INNER JOIN Marcas m ON m.dFecha_Jornada = a.dFecha_Asistencia AND m.nId_Colaborador = t.nId_Colaborador
        INNER JOIN inserted i ON i.nId_Solictud = m.nId_Solicitud
    WHERE  
        m.nTipo_Marcacion = 1
        AND i.nEstado_Solicitud = 2
        AND i.nId_Tipo_Solicitud = 4
        AND t.nId_Tipo_Entidad = 2 
        AND t.nEstado_Transaccion = 1;

    -- Inserción de nuevas transacciones, evitando duplicados
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
        -- Añadimos marca para que el otro trigger sepa que ya fue procesado
        i.sMotivo + ' [Procesado por trigger Solicitudes]',    
        CASE    
            WHEN i.nId_Tipo_Solicitud in(1,26,7,8) THEN i.nCantidad  -- Usar el valor de dCantidad_Minutos    
            WHEN i.nId_Tipo_Solicitud = 2 THEN (SELECT dbo.fn_total_min_days_by_schedule_Collaborator_V2(i.dFecha_Inicio, i.dFecha_Fin, i.nId_Colaborador))    
            ELSE NULL    
        END AS dCantidad_Minutos,    
        i.nUsuario_Creador,    
        i.dFecha_Inicio,    
        NULL,    
        i.dDatetime_Update,    
        i.nUsuario_Update,  
        CASE   
            WHEN i.nId_Tipo_Solicitud IN(1,2,7,8) THEN
                CASE 
                    WHEN i.nEstado_Solicitud IN (4,9) THEN 1
                    ELSE i.nEstado_Solicitud 
                END
            ELSE 3  
        END AS nEstado_Entidad  
    FROM inserted i    
    WHERE (    
        -- Acepta solicitudes con tipo = 8 (recuperacion) que sean aprobadas y pendientes   
        (i.nId_Tipo_Solicitud IN (1, 2, 7, 8) AND i.nEstado_Solicitud IN(1,3,4))  
        OR  
        (i.nId_Tipo_Solicitud = 26 AND i.nEstado_Solicitud = 3)
    )   
    -- Excluir las solicitudes tipo 4 con estado 2 (ya manejadas en el UPDATE)
    AND NOT (i.nId_Tipo_Solicitud = 4 AND i.nEstado_Solicitud = 2)
    -- Evitar inserción duplicada
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
        AND ts.nTipo_Transaccion = 1
    );
    
    -- Actualizar balances del colaborador
    EXEC sp_get_positive_negative_hours_by_Collaborator @dFecha = @Date, @sIds_Colaborador = @IdColaborador
END;
```