```SQL
CREATE TRIGGER dbo.tr_update_solicitudes_aprobadas_actualizar_rango_marcas
ON dbo.Solicitudes
AFTER UPDATE
AS
BEGIN
 SET NOCOUNT ON

DECLARE @dFecha DATE
DECLARE @nId_Colaborador INT
DECLARE @nId_Solicitud INT
DECLARE @Date DATE = GETDATE()

 INSERT INTO Xample (payload)
 SELECT (SELECT * FROM inserted FOR JSON AUTO);

-- Crear una tabla temporal para almacenar los datos de las solicitudes insertadas
SELECT
	i.nId_Solictud,
 i.dFecha_Inicio,
 i.nId_Colaborador
INTO #Temp_Solicitudes
FROM inserted i
JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = i.nId_Tipo_Solicitud
WHERE ts.nEstado = 1
 AND ts.nTipo_Frecuencia = 1
 AND ts.nAfecta_Asistencia = 1

	 WHILE EXISTS (SELECT 1 FROM #Temp_Solicitudes)
	 BEGIN
	 	SELECT TOP 1
	 	@dFecha = dFecha_Inicio,
	 	@nId_Colaborador = nId_Colaborador,
	 	@nId_Solicitud = nId_Solictud
	 	FROM #Temp_Solicitudes

	 	EXEC sp_Update_Range_Marks_by_Request @nId_Colaborador, @dFecha

	 	DELETE FROM #Temp_Solicitudes
 WHERE nId_Solictud = @nId_Solicitud;
	 END

EXEC sp_calcular_vacaciones @dFecha = @Date, @nUsuario_Update = 172
DROP TABLE #Temp_Solicitudes
END