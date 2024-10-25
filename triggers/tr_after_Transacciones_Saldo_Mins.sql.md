```sql
ALTER TRIGGER tr_after_Transacciones_Saldo_Mins 
ON Solicitudes
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    DECLARE @nId_Tipo_Entidad INT = 1;
    DECLARE @nId_Sub_Tipo_Entidad INT = 7;

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
        i.nEstado_Solicitud,
        @nId_Tipo_Entidad,
        @nId_Sub_Tipo_Entidad,
        d.nId_solictud,
        CASE 
            WHEN d.nId_Tipo_Solicitud = 1 THEN d.nCantidad
            WHEN d.nId_Tipo_Solicitud = 2 THEN d.nCantidad  -- Puedes ajustar esto si deseas usar una lógica diferente
            ELSE NULL
        END AS nTipo_Transaccion,
        d.sMotivo,
        CASE 
            WHEN d.nId_Tipo_Solicitud = 1 THEN 
                d.nCantidad  -- Mantiene la cantidad normal para tipo de solicitud 1
            WHEN d.nId_Tipo_Solicitud = 2 THEN 
                CASE 
                    WHEN d.nCantidad = 1 THEN 
                        CASE 
                            WHEN CAST(d.dFecha_Inicio AS DATE) = CAST(d.dFecha_Fin AS DATE) THEN 480
                            ELSE 960
                        END
                    WHEN d.nCantidad = 2 THEN 960
                    ELSE NULL
                END
            ELSE NULL
        END AS dCantidad_Minutos,
        d.nUsuario_Creador,
        d.dDatetime_Creador,
        NULL,
        d.dDatetime_Update,
        d.nUsuario_Update
    FROM inserted i
    JOIN deleted d ON i.nId_Solictud = d.nId_Solictud
    WHERE i.nEstado_Solicitud = 3
      AND (d.nEstado_Solicitud IS NULL OR d.nEstado_Solicitud <> 3);

    -- Eliminación de transacciones cuando nEstado_Solicitud cambia a 0
    DELETE FROM Transacciones_Saldo_Mins
    WHERE nId_Entidad IN (SELECT d.nId_Solictud 
                          FROM deleted d 
                          JOIN inserted i ON d.nId_Solictud = i.nId_Solictud
                          WHERE i.nEstado_Solicitud = 0);
END;


```