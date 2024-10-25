```sql
CREATE TRIGGER tr_after_Transacciones_Saldo_Mins
ON Solicitudes
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
	DECLARE @nId_Tipo_Entidad INT = 1
	DECLARE @nId_Sub_Tipo_Entidad INT =7
	
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
        d.nId_Colaborador,  -- Valor fijo o cambia según tu lógica
        d.nEstado,
        i.nEstado_Solicitud,
        @nId_Tipo_Entidad,
        @nId_Sub_Tipo_Entidad,
        d.nId_solictud,
         CASE 
            WHEN d.nId_Tipo_Solicitud IN (1, 2) THEN 2
            --WHEN d.nId_Tipo_Solicitud IN (6, 7, 8) THEN 1
            ELSE NULL -- o un valor por defecto si es necesario
        END AS nTipo_Transaccion,
        d.sMotivo,
        d.nCantidad,
        d.nUsuario_Creador,
        d.dDatetime_Creador,
        null,
        d.dDatetime_Update,
        d.nUsuario_Update
    FROM inserted i
    JOIN deleted d ON i.nId_Solictud  = d.nId_Solictud   -- Cambia por la clave correcta
     WHERE i.nEstado_Solicitud = 3  -- Solo si el nuevo estado es 3
      AND (d.nEstado_Solicitud IS NULL OR d.nEstado_Solicitud <> 3);
     
         -- Eliminación de transacciones cuando nEstado_Solicitud cambia a 0
    DELETE FROM Transacciones_Saldo_Mins
    WHERE nId_Entidad IN (SELECT d.nId_Solictud 
                          FROM deleted d 
                          JOIN inserted i ON d.nId_Solictud = i.nId_Solictud
                          WHERE i.nEstado_Solicitud = 0);
END;

```