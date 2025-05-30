```SQL
CREATE TRIGGER [dbo].[tr_after_Update_Attendance_Update_Tardanzas]
ON [dbo].[Asistencias]
AFTER UPDATE
AS
BEGIN
SET NOCOUNT ON;

DECLARE @nId_Usuario INT;
DECLARE @nId_Tipo_Entidad INT = 2; -- TARDANZAS
DECLARE @nId_Sub_Tipo_Entidad INT;
DECLARE @nId_Entidad INT;
DECLARE @dCantidad_Minutos DECIMAL(10, 2);
DECLARE @nEstado_Cierre INT = 1; -- PENDIENTE
DECLARE @nId_Colaborador INT;
DECLARE @nId_Horario INT;
DECLARE @nId_Last_Transaction INT;
DECLARE @dCantidad_Minutos_Last_Transaction DECIMAL(10, 2);
DECLARE @nTipo_Transaccion_Last_Transaction INT;
DECLARE @nEstado_Transaccion_Last_Transaction INT;
DECLARE @dFecha_Registro DATETIME;
DECLARE @nTipo_Marcacion INT = 1; -- PARA TARDANZAS DE MARCACIÓN DE ENTRADA


-- tabla temporal para procesar los datos
SELECT
i.nId_Asistencia,
i.nUsuario_Update,
i.dTiempo_Tardanza,
i.nId_Horario,
i.nId_Colaborador,
i.dDatetime_Creador,
i.dFecha_Asistencia
INTO #TempAsistencias
FROM INSERTED i


WHILE EXISTS (SELECT 1 FROM #TempAsistencias)
BEGIN


SELECT TOP 1
@nId_Entidad = nId_Asistencia,
@nId_Usuario = nUsuario_Update,
@dCantidad_Minutos = dTiempo_Tardanza,
@nId_Horario = nId_Horario,
@nId_Colaborador = nId_Colaborador,
@dFecha_Registro = dDatetime_Creador
FROM #TempAsistencias;

IF NOT EXISTS (SELECT TOP 1 a.nId_Cierre FROM cierreAsistencias a
                        JOIN cierreAsistenciaReportes car ON a.nId_Cierre = car.nId_Cierre
                        WHERE a.nId_Colaborador = @nId_Colaborador
                        AND CONVERT(DATE, @dFecha_Registro) BETWEEN car.dFecha_Inicio_Tardanza AND car.dFecha_Fin_Tardanza)
    BEGIN

-- Buscar la última transacción correspondiente a la asistencia
SELECT
@nId_Last_Transaction = nId_Transaccion,
@dCantidad_Minutos_Last_Transaction = ISNULL(dCantidad_Minutos, 0.00),
@nTipo_Transaccion_Last_Transaction = nTipo_Transaccion,
@nEstado_Transaccion_Last_Transaction = nEstado_Transaccion
FROM Transacciones_Saldo_Mins
WHERE nId_Entidad = @nId_Entidad
AND nEstado_Transaccion = 1 -- ACTIVO
ORDER BY nId_Transaccion DESC;

-- Si hay una transacción anterior con diferente cantidad de tardanza
IF @nId_Last_Transaction IS NOT NULL
BEGIN
IF @nEstado_Transaccion_Last_Transaction = 1 -- ACTIVO
AND @dCantidad_Minutos_Last_Transaction != @dCantidad_Minutos
	BEGIN
		UPDATE Transacciones_Saldo_Mins
		SET
		nEstado_Transaccion = 2, --CANCELADO
		sObservacion = 'Cancelada por actualización de asistencia',
		nUsuario_Update = @nId_Usuario,
		dDatetime_Update = DATEADD(hour,-5,GETDATE())
		WHERE nId_Entidad = @nId_Entidad
	END
END

-- Nueva suma si hay tardanza
IF @dCantidad_Minutos > 0.00
AND (@dCantidad_Minutos_Last_Transaction != @dCantidad_Minutos OR @nId_Last_Transaction IS NULL)
BEGIN
EXEC sp_Insert_Transacciones_Saldo_Mins_By_Tardanzas
@nId_Horario,
@nId_Usuario,
@nId_Entidad,
@dCantidad_Minutos,
@nId_Colaborador,
@dFecha_Registro,
@nTipo_Marcacion, -- PARA TARDANZAS DE MARCACIÓN DE ENTRADA
1; -- SUMA
END
    END


-- Eliminar la fila procesada de la tabla temporal
DELETE FROM #TempAsistencias
WHERE nId_Asistencia = @nId_Entidad;
END

-- Limpiar la tabla temporal
DROP TABLE #TempAsistencias;
END;