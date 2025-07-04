```SQL

CREATE PROCEDURE [dbo].[sp_closing_attendance_report_insert]
 @nId_Cierre INT OUTPUT,
 @dFecha_Inicio_Asistencia DATE,
 @dFecha_Fin_Asistencia DATE,
 @dFecha_Inicio_Tardanza DATE,
 @dFecha_Fin_Tardanza DATE,
 @dFecha_Periodo DATE,
 @nUsuario_Creador INT,
 @dDatetime_Creador DATETIME,
 @nUsuario_Update INT = NULL,
 @dDatetime_Update DATETIME = NULL,
 @nUsuario_Delete INT = NULL,
 @dDatetime_Delete DATETIME = NULL,
 @nEstado_Cierre INT = 2 -- precierre
AS
BEGIN
 INSERT INTO cierreAsistenciaReportes (
 dFecha_Inicio_Asistencia,
 dFecha_Fin_Asistencia,
 dFecha_Inicio_Tardanza,
 dFecha_Fin_Tardanza,
 dFecha_Periodo,
 nUsuario_Creador,
 dDatetime_Creador,
 nUsuario_Update,
 dDatetime_Update,
 nUsuario_Delete,
 dDatetime_Delete,
 nEstado_Cierre
 )
 VALUES (
 @dFecha_Inicio_Asistencia,
 @dFecha_Fin_Asistencia,
 @dFecha_Inicio_Tardanza,
 @dFecha_Fin_Tardanza,
 @dFecha_Periodo,
 @nUsuario_Creador,
 @dDatetime_Creador,
 @nUsuario_Update,
 @dDatetime_Update,
 @nUsuario_Delete,
 @dDatetime_Delete,
 @nEstado_Cierre
 );

 SET @nId_Cierre = SCOPE_IDENTITY();
END;