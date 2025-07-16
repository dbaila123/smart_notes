```SQL


CREATE   PROCEDURE [dbo].[sp_calculo_aprobacion_anulacion_vacaciones]
@nId_Solicitud int
AS
BEGIN
	--VARIABLES GENERALES
	DECLARE @nEstado_Solicitud INT = (SELECT TOP 1 nEstado_Solicitud FROM Solicitudes s where nId_Solictud = @nId_Solicitud);
	
	DECLARE @nCantidad decimal(18,2) = (SELECT TOP 1 nCantidad FROM Solicitudes s where nId_Solictud = @nId_Solicitud);
	
	DECLARE @nId_Colaborador INT = (SELECT TOP 1 nId_Colaborador FROM Solicitudes s where nId_Solictud = @nId_Solicitud)
	
	DECLARE @dDias_Vencidos_Laborales decimal(18,6) = (SELECT TOP 1 svc .dDias_Vencidos_Laborales FROM saldos_Vacaciones_Colaboradores svc WHERE nId_Colaborador = @nId_Colaborador)
	
	DECLARE @dDias_Ganados_Laborales decimal(18,6) = (SELECT TOP 1 svc .dDias_Ganados_Laborales FROM saldos_Vacaciones_Colaboradores svc WHERE nId_Colaborador = @nId_Colaborador)
	
	DECLARE @dDias_Generados_Laborales decimal(18,6) = (SELECT TOP 1 svc .dDias_Generados_Laborales FROM saldos_Vacaciones_Colaboradores svc WHERE nId_Colaborador = @nId_Colaborador)

	DECLARE @nTipo_Solicitud int = (SELECT TOP 1 nId_Tipo_Solicitud FROM Solicitudes s WHERE s.nId_Solictud = @nId_Solicitud);
	
	DECLARE @nUsuario_Update int = (SELECT TOP 1 nUsuario_Update FROM Solicitudes s WHERE s.nId_Solictud = @nId_Solicitud);

	IF @nTipo_Solicitud = 22 -- ADELANTO DE VACACIONES
	BEGIN
		IF @nEstado_Solicitud IN(1,3) -- PENDIENTE O APROBADO
		BEGIN
			UPDATE saldos_Vacaciones_Colaboradores
			SET dDias_Generados_Laborales = dDias_Generados_Laborales - @nCantidad,
				nUsuario_Update = @nUsuario_Update,
				dDatetime_Update = dbo.function_fecha_actual()
				WHERE nId_Colaborador = @nId_Colaborador
		END
		ELSE IF @nEstado_Solicitud IN(2,6) --ANULADO O RECHAZADO
		BEGIN
			UPDATE saldos_Vacaciones_Colaboradores
			SET dDias_Generados_Laborales = dDias_Generados_Laborales + @nCantidad,
				nUsuario_Update = @nUsuario_Update,
				dDatetime_Update = dbo.function_fecha_actual()
			WHERE nId_Colaborador = @nId_Colaborador
		END
	END 
	ELSE IF @nTipo_Solicitud IN(3,10) --VACACIONES Y VENTA DE VACACIONES
	BEGIN
		IF @nEstado_Solicitud IN(1,3) -- PENDIENTE O APROBADO
		BEGIN
			DECLARE @dDiferencia decimal = 0;
			DECLARE @dDiferencia_2 decimal = 0;
		
			set @dDiferencia = @dDias_Vencidos_Laborales - @nCantidad;
		
			IF @dDias_Vencidos_Laborales > 0
			BEGIN
				UPDATE saldos_Vacaciones_Colaboradores
				SET dDias_Vencidos_Laborales = (IIF(@dDiferencia < 0, 0, @dDiferencia)),
					nUsuario_Update = @nUsuario_Update,
					dDatetime_Update = dbo.function_fecha_actual()
				WHERE nId_Colaborador = @nId_Colaborador
				
			END
			IF @dDiferencia < 0
			BEGIN
				SET @dDiferencia_2 = @dDias_Ganados_Laborales + @dDiferencia
				UPDATE saldos_Vacaciones_Colaboradores
				SET dDias_Ganados_Laborales = IIF(@dDiferencia_2 < 0, 0, @dDiferencia_2),
					nUsuario_Update = @nUsuario_Update,
					dDatetime_Update = dbo.function_fecha_actual()
				WHERE nId_Colaborador = @nId_Colaborador
			END	
		END
		ELSE IF @nEstado_Solicitud IN(2,6) --ANULADO O RECHAZADO
		BEGIN
			DECLARE @nRegimen int = (SELECT TOP 1 r.nVacaciones_Dias_Laborales FROM Colaboradores c
									 JOIN Empresas_Usuarias eu
									 ON c.nId_Empresa = eu.nId_Empresa
									 JOIN Regimen r
									 ON r.nId_Regimen = eu.nId_Regimen
									 WHERE c.nId_Colaborador = @nId_Colaborador) 
	
			DECLARE @dDias_Ganados_Sum_Cantidad decimal = 0;
			
			DECLARE @dDiferencia_Dias_Ganados_Max_Dias_Ganados decimal = 0;
			
			SET @dDias_Ganados_Sum_Cantidad = @dDias_Ganados_Laborales + @nCantidad
			
			IF ROUND(@dDias_Ganados_Laborales,2) < @nRegimen
			BEGIN
				IF @dDias_Ganados_Sum_Cantidad > @nRegimen
				BEGIN
					SET @dDiferencia_Dias_Ganados_Max_Dias_Ganados = @dDias_Ganados_Sum_Cantidad - @nRegimen
					UPDATE saldos_Vacaciones_Colaboradores
					SET dDias_Vencidos_Laborales = @dDiferencia_Dias_Ganados_Max_Dias_Ganados,
						dDias_Ganados_Laborales = @nRegimen,
						nUsuario_Update = @nUsuario_Update,
						dDatetime_Update = dbo.function_fecha_actual()
					WHERE nId_Colaborador = @nId_Colaborador
				END
				ELSE
				BEGIN
					UPDATE saldos_Vacaciones_Colaboradores
					SET dDias_Ganados_Laborales = @dDias_Ganados_Sum_Cantidad,
						nUsuario_Update = @nUsuario_Update,
						dDatetime_Update = dbo.function_fecha_actual()
					WHERE nId_Colaborador = @nId_Colaborador
				END
			END
			ELSE 
			BEGIN
				UPDATE saldos_Vacaciones_Colaboradores
				SET dDias_Vencidos_Laborales = dDias_Vencidos_Laborales + @nCantidad,
					nUsuario_Update = @nUsuario_Update,
					dDatetime_Update = dbo.function_fecha_actual()
				WHERE nId_Colaborador = @nId_Colaborador
			END
		END
	END
END
```