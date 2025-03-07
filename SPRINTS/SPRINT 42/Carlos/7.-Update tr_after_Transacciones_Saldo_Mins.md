	```sql

CREATE TRIGGER tr_after_Transacciones_Saldo_Mins
ON Solicitudes
AFTER UPDATE, INSERT
AS
BEGIN
    SET NOCOUNT ON;
    -- Eliminación de transacciones cuando nEstado_Solicitud YA NO ES APROBADO
    DELETE FROM Transacciones_Saldo_Mins
    WHERE nId_Entidad
 IN (
        SELECT i.nId_Solictud
        FROM INSERTED i
        WHERE i.nEstado_Solicitud != 3
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
        nUsuario_Update
    )
  
   SELECT
        i.nId_Colaborador,
        i.nEstado,
        1,
        1 AS nId_Tipo_Entidad,
        CASE
            WHEN i.nId_Tipo_Solicitud = 26 THEN (SELECT nId_Entidad FROM Entidades_Transacciones WHERE sDescripcion = 'SOLICITUDES VENCIDAS')
  
          ELSE (SELECT dbo.fn_nEntidad_Transaction_By_Request_Date (i.dFecha_Inicio, CONVERT(DATE, GETDATE())))
        END AS  nId_Sub_Tipo_Entidad,
        i.nId_solictud,
        1 AS nTipo_Transaccion,
        i.sMotivo,
        CASE
            WHEN i.nId_Tipo_Solicitud in(1,26) THEN i.nCantidad  -- Usar el valor de dCantidad_Minutos
            WHEN i.nId_Tipo_Solicitud = 2 THEN (SELECT  dbo.fn_total_min_days_by_schedule_Collaborator_V2(i.dFecha_Inicio, i.dFecha_Fin, i.nId_Colaborador))
            
ELSE NULL
        END AS dCantidad_Minutos,
        i.nUsuario_Creador,
        i.dFecha_Inicio,
        NULL,
        i.dDatetime_Update,
        i.nUsuario_Update
    FROM inserted i
    WHERE i.nEstado_Solicitud = 3
    AND i.nId_Tipo_Solicitud IN (1,2,26)
END;

```