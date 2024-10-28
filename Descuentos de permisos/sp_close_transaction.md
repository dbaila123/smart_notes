```sql
CREATE PROCEDURE sp_close_transaction
    @nId_Transaccion INT,
    @nId_Colaborador INT,
    @dFecha_Cierre DATETIME,
    @nUsuario_Cerrador INT
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @dCantidad_Minutos INT;
    DECLARE @jsonTransacciones NVARCHAR(MAX);

    -- Obtener la cantidad total de minutos descontados desde la vista por colaborador
    SELECT @dCantidad_Minutos = SUM(CantidadTotalDescontado)
    FROM v_listado_Transacciones_Saldo_Mins
    WHERE nId_Colaborador = @nId_Colaborador;

    -- Obtener las transacciones en formato JSON
    EXEC sp_smart_get_descuentos_solicitudes @nId_Colaborador, @nId_Colaborador;

    SELECT @jsonTransacciones = (
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
            CONVERT(VARCHAR(8), s.dHora_Inicio, 108) AS dHora_Inicio,
            CONVERT(VARCHAR(8), s.dHora_Fin, 108) AS dHora_Fin,
            s.sMotivo,
            s.nId_Supervisor,
            s.nId_Tipo_Motivo,
            s.sObservacion_Lider
        FROM Transacciones_Saldo_Mins tsm
        JOIN Solicitudes s ON tsm.nId_Entidad = s.nId_Solictud
        JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador 
        JOIN Personas p ON p.nId_Persona = c.nId_Persona 
        JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud 
        WHERE tsm.nId_Colaborador = @nId_Colaborador
          AND ts.b_recuperable = 1
          AND tsm.nId_Cierre IS NULL
        FOR JSON PATH
    );

    -- Actualizar la transacción especificada
    UPDATE Transacciones_Saldo_Mins
    SET 
        nId_Cierre = @nId_Transaccion,
        dCantidad_Minutos = 0,
        dDatetime_Update = @dFecha_Cierre,
        nUsuario_Update = @nUsuario_Cerrador
    WHERE 
        nId_Colaborador = @nId_Colaborador
        AND nId_Transaccion = @nId_Transaccion;

    -- Verificar si la actualización fue exitosa
    IF @@ROWCOUNT = 0
    BEGIN
        RAISERROR('No se encontró la transacción o no se pudo actualizar.', 16, 1);
        RETURN;
    END

    -- Insertar en la tabla de historial de cierre
    INSERT INTO Historico_Cierre_Transacciones 
    (
        nId_Cierre, 
        dDatetime_Creacion, 
        nUsuario_Creador, 
        dCantidad_Minutos,
        Json_Detalles_Transacciones
    )
    VALUES 
    (
        @nId_Transaccion,
        @dFecha_Cierre,
        @nUsuario_Cerrador,
        @dCantidad_Minutos,   -- Cantidad total de minutos descontados
        @jsonTransacciones
    );

END;
```