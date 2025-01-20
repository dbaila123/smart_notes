--realizar copias de las siguiente tablas 
--select * into Transacciones_Saldo_Mins

--realizar truncate
--truncate table Transacciones_Saldo_Mins
--truncate table closure_Request
--truncate table cierreAsistencias
--truncate table cierreAsistenciaReportes

--por si causan inconvenientes
--ALTER TABLE tenant0001.dbo.cierreAsistencias 
--DROP CONSTRAINT FK_cierreAsistencias_CierreAsistenciaReporte_nId_Cierre;

--TRUNCATE TABLE tenant0001.dbo.cierreAsistenciaReportes;

--ALTER TABLE tenant0001.dbo.cierreAsistencias
--ADD CONSTRAINT FK_cierreAsistencias_CierreAsistenciaReporte_nId_Cierre
--FOREIGN KEY (nId_Cierre)
--REFERENCES tenant0001.dbo.cierreAsistenciaReportes(nId_Cierre);

--deshabilitar tareo
--EXEC sp_disable_or_enable_all_triggers_by_table Tareas, enable

--revisar tareas en el excel de consolidado de horas verificar id's en el sistema y ver la cantidad de generar cierre de pago desde el sistema
--despues de pasar las tareas a pago desde el sistema actualizar el estado de pago de las horas del  cierre 11 y 12
--primero validar la cantidad de tareas que se hayan pagado

--select count(*) from tareas where nId_Cierre = 

--update tareas set nEstado_Pago = 2 where nId_Cierre IN (11,12)

--cierre asistencia
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-01-01 00:00:00',@dFecha_Fin_Asistencia='2024-01-31 00:00:00',@dFecha_Inicio_Tardanza='2024-01-01 00:00:00',@dFecha_Fin_Tardanza='2024-01-31 00:00:00',@dFecha_Periodo='2024-01-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-02-01 00:00:00',@dFecha_Fin_Asistencia='2024-02-29 00:00:00',@dFecha_Inicio_Tardanza='2024-02-01 00:00:00',@dFecha_Fin_Tardanza='2024-02-29 00:00:00',@dFecha_Periodo='2024-02-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-03-01 00:00:00',@dFecha_Fin_Asistencia='2024-03-25 00:00:00',@dFecha_Inicio_Tardanza='2024-03-01 00:00:00',@dFecha_Fin_Tardanza='2024-03-25 00:00:00',@dFecha_Periodo='2024-03-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-03-26 00:00:00',@dFecha_Fin_Asistencia='2024-04-29 00:00:00',@dFecha_Inicio_Tardanza='2024-03-26 00:00:00',@dFecha_Fin_Tardanza='2024-04-29 00:00:00',@dFecha_Periodo='2024-04-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-04-30 00:00:00',@dFecha_Fin_Asistencia='2024-05-30 00:00:00',@dFecha_Inicio_Tardanza='2024-04-30 00:00:00',@dFecha_Fin_Tardanza='2024-05-30 00:00:00',@dFecha_Periodo='2024-05-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-05-31 00:00:00',@dFecha_Fin_Asistencia='2024-06-27 00:00:00',@dFecha_Inicio_Tardanza='2024-05-31 00:00:00',@dFecha_Fin_Tardanza='2024-06-27 00:00:00',@dFecha_Periodo='2024-06-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-06-28 00:00:00',@dFecha_Fin_Asistencia='2024-07-30 00:00:00',@dFecha_Inicio_Tardanza='2024-06-28 00:00:00',@dFecha_Fin_Tardanza='2024-07-30 00:00:00',@dFecha_Periodo='2024-07-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-07-31 00:00:00',@dFecha_Fin_Asistencia='2024-08-28 00:00:00',@dFecha_Inicio_Tardanza='2024-07-31 00:00:00',@dFecha_Fin_Tardanza='2024-08-28 00:00:00',@dFecha_Periodo='2024-08-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-08-29 00:00:00',@dFecha_Fin_Asistencia='2024-09-27 00:00:00',@dFecha_Inicio_Tardanza='2024-08-29 00:00:00',@dFecha_Fin_Tardanza='2024-09-27 00:00:00',@dFecha_Periodo='2024-09-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-09-28 00:00:00',@dFecha_Fin_Asistencia='2024-10-29 00:00:00',@dFecha_Inicio_Tardanza='2024-09-28 00:00:00',@dFecha_Fin_Tardanza='2024-10-29 00:00:00',@dFecha_Periodo='2024-10-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-10-30 00:00:00',@dFecha_Fin_Asistencia='2024-11-27 00:00:00',@dFecha_Inicio_Tardanza='2024-10-30 00:00:00',@dFecha_Fin_Tardanza='2024-11-27 00:00:00',@dFecha_Periodo='2024-11-01 00:00:00'
--exec sp_create_closing_attendance @nId_User=255,@dFecha_Inicio_Asistencia='2024-11-28 00:00:00',@dFecha_Fin_Asistencia='2024-12-26 00:00:00',@dFecha_Inicio_Tardanza='2024-11-28 00:00:00',@dFecha_Fin_Tardanza='2024-12-26 00:00:00',@dFecha_Periodo='2024-12-01 00:00:00'

