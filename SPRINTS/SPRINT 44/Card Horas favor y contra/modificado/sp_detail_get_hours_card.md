```sql
CREATE OR ALTER PROCEDURE sp_detail_get_hours_card
    @nId_Colaborador INT,
    @dFecha DATE
AS
BEGIN
    SET NOCOUNT ON;

	DECLARE @Resultados TABLE (
        sCategoria VARCHAR(50),
        sOrigen VARCHAR(50),
        nId_Colaborador INT,
        nId_Entidad
 INT,
        dCantidad_Minutos INT,
		dFecha_Creacion DATETIME,
		sLink_Notificacion NVARCHAR(MAX) NULL,
		nCodigo INT
    );

    -- BOLSA ANTERIOR
	INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nId_Entidad, dCantidad_Minutos, dFecha_Creacion, nCodigo)
    SELECT 
	   'BOLSA' AS sCategoria,
	   'Saldo de cierre anterior' AS sOrigen,
        tsm.nId_Colaborador, 
        tsm.nId_Entidad AS nId_Solicitud,
        tsm.dCantidad_Minutos,
		tsm.dDatetime_Creacion,
		1
    FROM transacciones_saldo_mins tsm
    WHERE tsm.nId_Tipo_Entidad = 5
        AND tsm.nId_Sub_Tipo_Entidad = 8 
        AND tsm.nId_Cierre IS NULL 
        AND tsm.nId_Colaborador = @nId_Colaborador
        AND CONVERT(DATE, tsm.dDatetime_Creacion) <= @dFecha;
	IF NOT EXISTS

 (SELECT 1 FROM @Resultados WHERE sCategoria = 'BOLSA')
	 BEGIN
        INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nCodigo)
        VALUES ('BOLSA', 'Saldo de cierre anterior', @nId_Colaborador, 1);
    END

    -- EXTRAS
	INSERT INTO 
@Resultados (sCategoria, sOrigen, nId_Colaborador, nId_Entidad, dCantidad_Minutos, dFecha_Creacion, nCodigo)
SELECT
		'SOLICITUDES' AS sCategoria,
		'Extras' AS sOrigen,
        t.nId_Colaborador,
        s.nId_Solictud,
        SUM(nMinutos) AS nMinutos,


		t.dFecha_Registro,
		2
    FROM Tareas t
	JOIN Solicitudes s ON t.nId_Tarea = s.nId_Tarea
    WHERE t.sTipo_Hora = 2
        AND t.nEstado_Tarea = 3
        AND t.nEstado_Pago IS NULL
		AND s.nEstado_Solicitud = 3
        AND t.nId_Colaborador = @nId_Colaborador
        AND t.dFecha_Registro <= @dFecha
    GROUP BY t.nId_Colaborador, s.nId_Solictud, t.dFecha_Registro;

	 IF NOT EXISTS (SELECT 1 FROM @Resultados WHERE sCategoria = 'SOLICITUDES')
    BEGIN
        INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nCodigo)
        VALUES ('SOLICITUDES', 'Extras', @nId_Colaborador, 2);
    END


    -- COMPENSACIÓN/RECUPERACIÓN
	INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nId_Entidad, dCantidad_Minutos, dFecha_Creacion, nCodigo)
    SELECT
		'SOLICITUDES' AS sCategoria,
		'Compensación / Recuperación' AS sOrigen,
        tsm.nId_Colaborador,
        t.nId_Solictud AS nId_Tarea,
        tsm.dCantidad_Minutos,
		tsm.dDatetime_Creacion,
		3
    FROM Transacciones_Saldo_Mins tsm

 
   JOIN Solicitudes t ON tsm.nId_Entidad = t.nId_Tarea
    WHERE tsm.nId_Tipo_Entidad = 9
        AND tsm.nId_Sub_Tipo_Entidad = 8
        AND tsm.nId_Cierre IS NULL
        AND t.nEstado_Solicitud IN(1,3)
        AND CONVERT(DATE, tsm.dDatetime_Creacion) <= @dFecha
        AND tsm.nId_Colaborador = @nId_Colaborador;

	IF NOT EXISTS (SELECT 1 FROM @Resultados WHERE sCategoria = 'SOLICITUDES' AND sOrigen = 'Compensación / Recuperación')
    BEGIN
        INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nCodigo)
        VALUES ('SOLICITUDES', 'Compensación / Recuperación', @nId_Colaborador, 3);
    END

    -- PERMISOS
	INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nId_Entidad, dCantidad_Minutos, dFecha_Creacion, nCodigo)
    SELECT
	
	'SOLICITUDES' AS sCategoria,
		'Permisos' AS sOrigen,
        tsm.nId_Colaborador,
        tsm.nId_Entidad AS nId_Solicitud,
        SUM(CASE WHEN tsm.nTipo_Transaccion = 1
                 AND tsm.nId_Sub_Tipo_Entidad IN (6,7)
                 AND tsm.nId_Cierre IS NULL
                 THEN tsm.dCantidad_Minutos ELSE 0 END) AS nPermisos,
		tsm.dDatetime_Creacion,
		4
    FROM Transacciones_Saldo_Mins tsm
    WHERE tsm.nEstado_Transaccion = 1
        AND tsm.nId_Tipo_Entidad = 1
        AND CONVERT(DATE, tsm.dDatetime_Creacion) <= @dFecha
        AND tsm.nId_Colaborador = @nId_Colaborador
    GROUP BY tsm.nId_Colaborador, tsm.nId_Entidad, tsm.dDatetime_Creacion
    HAVING SUM(CASE WHEN tsm.nTipo_Transaccion = 1
                    AND tsm.nId_Sub_Tipo_Entidad IN (6,7)
                    AND tsm.nId_Cierre IS NULL
                    THEN tsm.dCantidad_Minutos ELSE 0 END) > 0
    ORDER BY tsm.nId_Entidad;
	IF NOT EXISTS (SELECT 1 FROM @Resultados WHERE sCategoria = 'SOLICITUDES' AND sOrigen = 'Permisos'
)
    BEGIN
        INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nCodigo)
        VALUES ('SOLICITUDES', 'Permisos', @nId_Colaborador, 4);
    END

    -- TARDANZA CON TOLERANCIA
    INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nId_Entidad, dCantidad_Minutos, dFecha_Creacion, nCodigo)
    SELECT
		'ASISTENCIAS' AS sCategoria,
		'Tardanzas recuperables' AS sOrigen,
        tsm.nId_Colaborador,
        tsm.nId_Entidad,
        SUM(CASE WHEN tsm.nTipo_Transaccion = 1
     

            AND tsm.nId_Sub_Tipo_Entidad = 3
                 AND tsm.nId_Cierre IS NULL
                 THEN tsm.dCantidad_Minutos ELSE 0 END) -
        SUM(CASE WHEN tsm.nTipo_Transaccion = 2
                 AND tsm.nId_Sub_Tipo_Entidad = 3
          

       AND tsm.nId_Cierre IS NULL
                 THEN tsm.dCantidad_Minutos ELSE 0 END) AS dTardanza_Con_Tolerancia,
		tsm.dDatetime_Creacion,
		5
    FROM Transacciones_Saldo_Mins tsm
    LEFT JOIN Asistencias a ON a.nId_Asistencia = tsm.nId_Entidad
  

  WHERE tsm.nEstado_Transaccion = 1
        AND tsm.nId_Tipo_Entidad = 2
        AND CONVERT(DATE, a.dFecha_Asistencia) <= @dFecha
        AND tsm.nId_Colaborador = @nId_Colaborador
    GROUP BY tsm.nId_Colaborador, tsm.nId_Entidad, tsm.dDatetime_Creacion


    HAVING SUM(CASE WHEN tsm.nTipo_Transaccion = 1
                    AND tsm.nId_Sub_Tipo_Entidad = 3
                    AND tsm.nId_Cierre IS NULL
                    THEN tsm.dCantidad_Minutos ELSE 0 END) -
           SUM(CASE WHEN tsm.nTipo_Transaccion = 2
                    AND tsm.nId_Sub_Tipo_Entidad = 3
                    AND tsm.nId_Cierre IS NULL
                    THEN tsm.dCantidad_Minutos ELSE 0 END) > 0
    ORDER BY tsm.nId_Entidad;
	IF NOT EXISTS (SELECT 1 FROM @Resultados WHERE sCategoria = 'ASISTENCIAS' AND sOrigen = 'Tardanzas recuperables')
    BEGIN
        INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nCodigo)
        VALUES ('ASISTENCIAS', 'Tardanzas recuperables', @nId_Colaborador, 5);
    END

    -- TARDANZA SIN TOLERANCIA
    INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nId_Entidad, dCantidad_Minutos, dFecha_Creacion, nCodigo)
    SELECT
		'ASISTENCIAS' AS sCategoria,
		'Tardanzas no recuperables' AS sOrigen,
        tsm.nId_Colaborador,
 

       tsm.nId_Entidad,
        SUM(CASE WHEN tsm.nTipo_Transaccion = 1
                 AND tsm.nId_Sub_Tipo_Entidad = 4
                 AND tsm.nId_Cierre IS NULL
                 THEN tsm.dCantidad_Minutos ELSE 0 END) -
        SUM(CASE WHEN tsm.nTipo_Transaccion = 2
                 AND tsm.nId_Sub_Tipo_Entidad = 4
                 AND tsm.nId_Cierre IS NULL
                 THEN tsm.dCantidad_Minutos ELSE 0 END) AS dTardanza_Sin_Tolerancia,
		a.dDatetime_Creador,
		6
    FROM Transacciones_Saldo_Mins tsm
    LEFT JOIN Asistencias a ON a.nId_Asistencia = tsm.nId_Entidad
    WHERE tsm.nEstado_Transaccion = 1
        AND tsm.nId_Tipo_Entidad = 2
        AND CONVERT(DATE, a.dFecha_Asistencia) <= @dFecha
        AND tsm.nId_Colaborador = @nId_Colaborador


    GROUP BY tsm.nId_Colaborador, tsm.nId_Entidad, a.dDatetime_Creador
    HAVING SUM(CASE WHEN tsm.nTipo_Transaccion = 1
                    AND tsm.nId_Sub_Tipo_Entidad = 4
                    AND tsm.nId_Cierre IS NULL
                    THEN tsm.dCantidad_Minutos ELSE 0 END) -
           SUM(CASE WHEN tsm.nTipo_Transaccion = 2
                    AND tsm.nId_Sub_Tipo_Entidad = 4
                    AND tsm.nId_Cierre IS NULL
                    THEN tsm.dCantidad_Minutos ELSE 0 END) > 0
    ORDER BY
 tsm.nId_Entidad;
	IF NOT EXISTS (SELECT 1 FROM @Resultados WHERE sCategoria = 'ASISTENCIAS' AND sOrigen = 'Tardanzas no recuperables')
    BEGIN
        INSERT INTO @Resultados (sCategoria, sOrigen, nId_Colaborador, nCodigo)
        VALUES ('ASISTENCIAS', 'Tardanzas no recuperables', @nId_Colaborador, 6);
    END

    SELECT * FROM @Resultados ORDER BY sCategoria, nId_Entidad;
END;
```