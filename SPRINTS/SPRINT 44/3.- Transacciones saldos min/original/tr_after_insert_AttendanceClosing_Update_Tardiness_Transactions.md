```sql
CREATE OR ALTER TRIGGER [dbo].[tr_after_insert_AttendanceClosing_Update_Tardiness_Transactions]  
ON [dbo].[cierreAsistenciaReportes]  
AFTER INSERT  
AS  
BEGIN  
    SET NOCOUNT ON;  
  
    DECLARE @nId_Usuario INT;  
    DECLARE @dFecha_Inicio DATE;  
    DECLARE @dFecha_Fin DATE;  
    DECLARE @nId_Cierre INT;  
    DECLARE @tTransacciones TABLE(nId_Transaccion INT);  
    DECLARE @dDatetime_Update DATETIME = DATEADD(HOUR, -5, GETDATE());  
    DECLARE @sJson_Old NVARCHAR(MAX);  
  
    -- Captura los datos en formato JSON desde la tabla DELETED  
    SELECT @sJson_Old = (  
        SELECT  
            *  
        FROM  
            INSERTED  
        FOR JSON PATH, WITHOUT_ARRAY_WRAPPER  
    );  
  
    -- Captura los datos de la tabla INSERTED  
    SELECT  
        @nId_Cierre = nId_Cierre,  
        @dFecha_Inicio = dFecha_Inicio_Tardanza,  
        @dFecha_Fin = dFecha_Fin_Tardanza,  
        @nId_Usuario = nUsuario_Creador  
    FROM INSERTED;  
  
    -- Inserta los identificadores de transacciones en una tabla de variables  
    INSERT INTO @tTransacciones (nId_Transaccion)  
    SELECT  
        tsm.nId_Transaccion  
    FROM  
        Transacciones_Saldo_Mins tsm  
    JOIN  
        Asistencias a ON tsm.nId_Entidad = a.nId_Asistencia  
    WHERE  
        a.dFecha_Asistencia BETWEEN @dFecha_Inicio AND @dFecha_Fin  
        AND tsm.nEstado_Transaccion = 1;  
  
    -- Actualiza las transacciones seleccionadas  
    UPDATE Transacciones_Saldo_Mins  
    SET  
        nId_Cierre = @nId_Cierre,  
        nUsuario_Update = @nId_Usuario,  
        dDatetime_Update = @dDatetime_Update  
    WHERE  
        nId_Transaccion IN (SELECT nId_Transaccion FROM @tTransacciones);  
  
    -- Inserta el registro en la tabla History_Hours_Closing  
    INSERT INTO History_Hours_Closing (nId_Entidad, nTipo_Historico, sJson_Old, dDatetime_Creador, nUsuario_Creador)  
    VALUES  
    (  
        @nId_Cierre,  
        0, -- creacion  
        @sJson_Old,  
        @dDatetime_Update,  
        @nId_Usuario  
    );  


END;
```
