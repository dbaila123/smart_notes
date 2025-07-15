```SQL
CREATE      PROCEDURE [dbo].[sp_calcular_vacaciones]
@dFecha DATETIME,
@nUsuario_Update int
AS
BEGIN
	BEGIN 
	----------------------------
	declare @saldos_vacaciones Table(
										nId_Colaborador int, 
										dDias_Ganados_Laborales_old decimal(18,6),
										dDias_Ganados_Laborales_new decimal(18,6),
										dDias_Ganados_Laborales_has_changed int,
										dDias_Vencidos_Laborales_old decimal(18,6),
										dDias_Vencidos_Laborales_new decimal(18,6),
										dDias_Vencidos_Laborales_has_changed int,
										dDias_Generados_Laborales_old decimal(18,16),
										dDias_Generados_Laborales_new decimal(18,16),
										dDias_Generados_Laborales_has_changed int,
										dDias_Ganados_Calendario_old decimal(18,6),
										dDias_Ganados_Calendario_new decimal(18,6),
										dDias_Ganados_Calendario_has_changed int,
										dDias_Vencidos_Calendario_old decimal(18,6),
										dDias_Vencidos_Calendario_new decimal(18,6),
										dDias_Vencidos_Calendario_has_changed int,
										dDias_Generados_Calendario_old decimal(18,6),
										dDias_Generados_Calendario_new decimal(18,6),
										dDias_Generados_Calendario_has_changed int)
										
										
	INSERT INTO @saldos_vacaciones
	select 
		nId_Colaborador, 
		dDias_Ganados_Laborales dDias_Ganados_Laborales_old,
		0 dDias_Ganados_Laborales_new,
		0 dDias_Ganados_Laborales_has_changed,
		dDias_Vencidos_Laborales dDias_Vencidos_Laborales_old,
		0 dDias_Vencidos_Laborales_new,
		0 dDias_Vencidos_Laborales_has_changed,
		dDias_Generados_Laborales dDias_Generados_Laborales_old,
		0 dDias_Generados_Laborales_new,
		0 dDias_Generados_Laborales_has_changed,
		dDias_Ganados_Calendario dDias_Ganados_Calendario_old,
		0 dDias_Ganados_Calendario_new,
		0 dDias_Ganados_Calendario_has_changed,
		dDias_Vencidos_Calendario dDias_Vencidos_Calendario_old,
		0 dDias_Vencidos_Calendario_new,
		0 dDias_Vencidos_Calendario_has_changed,
		dDias_Generados_Calendario dDias_Generados_Calendario_old,
		0 dDias_Generados_Calendario_new,
		0 dDias_Generados_Calendario_has_changed
	FROM 
		saldos_Vacaciones_Colaboradores
		
	
	UPDATE sv
	SET
		---------CALCULO PARA DIAS LABORALES
		sv.dDias_Ganados_Laborales = CASE WHEN
									 CONVERT(DATE,@dFecha) = CONVERT(DATE,sv.dUltimo_Periodo_Ganado)
									 THEN
										sv.dDias_Ganados_Laborales - sv.dDias_Ganados_Laborales
									 ELSE
										sv.dDias_Ganados_Laborales
									 END,
		sv.dDias_Vencidos_Laborales = CASE WHEN
									  CONVERT(DATE,@dFecha) = CONVERT(DATE,sv.dUltimo_Periodo_Ganado)
									  THEN
										sv.dDias_Vencidos_Laborales + sv.dDias_Ganados_Laborales
									  ELSE
										sv.dDias_Vencidos_Laborales
									  END,
		-----------CALCULO PARA DIAS CALENDARIO
		sv.dDias_Ganados_Calendario = CASE WHEN
									 CONVERT(DATE,@dFecha) = CONVERT(DATE,sv.dUltimo_Periodo_Ganado)
									 THEN
										sv.dDias_Ganados_Calendario - sv.dDias_Ganados_Calendario
									 ELSE
										sv.dDias_Ganados_Calendario
									 END,
		sv.dDias_Vencidos_Calendario = CASE WHEN
									  CONVERT(DATE,@dFecha) = CONVERT(DATE,sv.dUltimo_Periodo_Ganado)
									  THEN
										sv.dDias_Vencidos_Calendario + sv.dDias_Ganados_Calendario
									  ELSE
										sv.dDias_Vencidos_Calendario
									  END,
		sv.nUsuario_Update = @nUsuario_Update,
		sv.dDatetime_Update = dbo.function_fecha_actual()
	FROM saldos_Vacaciones_Colaboradores AS sv;
	END	 
	
	BEGIN
	UPDATE sv
	SET sv.dDias_Generados_Laborales = CASE WHEN
									   CONVERT(DATE,@dFecha) = (SELECT DATEADD(MONTH , DATEDIFF(month, dFec_Ingreso ,@dFecha), CONVERT(DATE,dFec_Ingreso)) aniversario_mensual from Colaboradores c where c.nId_Colaborador = sv.nId_Colaborador)
									   AND CONVERT(DATE,@dFecha) > (SELECT TOP 1 CONVERT(DATE,c2.dFec_Ingreso) from Colaboradores c2 where c2.nId_Colaborador = sv.nId_Colaborador)
									   THEN
											sv.dDias_Generados_Laborales + (SELECT TOP 1 (r.nVacaciones_Dias_Laborales)/12.00000000000 FROM Colaboradores c
																		    JOIN Empresas_Usuarias eu
																			ON c.nId_Empresa = eu.nId_Empresa
																		    JOIN Regimen r
																			ON r.nId_Regimen = eu.nId_Regimen
																			WHERE c.nId_Colaborador = sv.nId_Colaborador) -- Abono mensual
									   ELSE
											sv.dDias_Generados_Laborales
									   END,
		-----------DIAS CALENDARIO
		sv.dDias_Generados_Calendario = CASE WHEN
									   CONVERT(DATE,@dFecha) = (SELECT DATEADD(MONTH , DATEDIFF(month, dFec_Ingreso ,@dFecha), CONVERT(DATE,dFec_Ingreso)) aniversario_mensual from Colaboradores c where c.nId_Colaborador = sv.nId_Colaborador)
									   AND CONVERT(DATE,@dFecha) > (SELECT TOP 1 CONVERT(DATE,c2.dFec_Ingreso) from Colaboradores c2 where c2.nId_Colaborador = sv.nId_Colaborador)
									   THEN
											sv.dDias_Generados_Calendario + (SELECT TOP 1 (r.nVacaciones_Dias_Calendario)/12.00000000000 FROM Colaboradores c
																		     JOIN Empresas_Usuarias eu
																			 ON c.nId_Empresa = eu.nId_Empresa
																		     JOIN Regimen r
																			 ON r.nId_Regimen = eu.nId_Regimen
																			 WHERE c.nId_Colaborador = sv.nId_Colaborador) -- Abono mensual
									   ELSE
											sv.dDias_Generados_Calendario
									   END,
		sv.nUsuario_Update = @nUsuario_Update,
		sv.dDatetime_Update = dbo.function_fecha_actual()
	FROM saldos_Vacaciones_Colaboradores AS sv;
	END  
	
	BEGIN 
	UPDATE sv
	SET sv.dDias_Ganados_Laborales = CASE WHEN
								     CONVERT(DATE,@dFecha) = (SELECT DATEADD(YEAR , DATEDIFF(YEAR , dFec_Ingreso ,@dFecha), CONVERT(DATE,dFec_Ingreso)) aniversario_anual from Colaboradores c where c.nId_Colaborador = sv.nId_Colaborador)
									 AND CONVERT(DATE,@dFecha) > (SELECT TOP 1 CONVERT(DATE,c2.dFec_Ingreso) from Colaboradores c2 where c2.nId_Colaborador = sv.nId_Colaborador)
									 THEN
										    sv.dDias_Generados_Laborales + sv.dDias_Ganados_Laborales
									 ELSE
											sv.dDias_Ganados_Laborales
									 END,
		sv.dDias_Generados_Laborales = CASE WHEN
										CONVERT(DATE,@dFecha) = (SELECT DATEADD(YEAR  , DATEDIFF(YEAR , dFec_Ingreso ,@dFecha), CONVERT(DATE,dFec_Ingreso)) aniversario_mensual from Colaboradores c where c.nId_Colaborador = sv.nId_Colaborador)
										AND CONVERT(DATE,@dFecha) > (SELECT TOP 1 CONVERT(DATE,c2.dFec_Ingreso) from Colaboradores c2 where c2.nId_Colaborador = sv.nId_Colaborador)
										THEN
											sv.dDias_Generados_Laborales - sv.dDias_Generados_Laborales
										ELSE
											sv.dDias_Generados_Laborales
										END,
		---------CALCULO DIAS CALENDARIO
		sv.dDias_Ganados_Calendario = CASE WHEN
								     CONVERT(DATE,@dFecha) = (SELECT DATEADD(YEAR , DATEDIFF(YEAR , dFec_Ingreso ,@dFecha), CONVERT(DATETIME,dFec_Ingreso)) aniversario_anual from Colaboradores c where c.nId_Colaborador = sv.nId_Colaborador)
									 AND @dFecha > (SELECT TOP 1 CONVERT(DATE,c2.dFec_Ingreso) from Colaboradores c2 where c2.nId_Colaborador = sv.nId_Colaborador)
									 THEN
										    sv.dDias_Generados_Calendario + sv.dDias_Ganados_Calendario
									 ELSE
											sv.dDias_Ganados_Calendario
									 END,
		sv.dDias_Generados_Calendario = CASE WHEN
										CONVERT(DATE,@dFecha) = (SELECT DATEADD(YEAR  , DATEDIFF(YEAR , dFec_Ingreso ,@dFecha), CONVERT(DATETIME,dFec_Ingreso)) aniversario_mensual from Colaboradores c where c.nId_Colaborador = sv.nId_Colaborador)
										AND @dFecha > (SELECT TOP 1 CONVERT(DATE,c2.dFec_Ingreso) from Colaboradores c2 where c2.nId_Colaborador = sv.nId_Colaborador)
										THEN
											sv.dDias_Generados_Calendario - sv.dDias_Generados_Calendario
										ELSE
											sv.dDias_Generados_Calendario
										END,
		sv.dUltimo_Periodo_Ganado = CASE WHEN
									CONVERT(DATE,@dFecha) = (SELECT DATEADD(YEAR  , DATEDIFF(YEAR , dFec_Ingreso ,@dFecha), CONVERT(DATE,dFec_Ingreso)) aniversario_mensual from Colaboradores c where c.nId_Colaborador = sv.nId_Colaborador)
									AND CONVERT(DATE,@dFecha) > (SELECT TOP 1 CONVERT(DATE,c2.dFec_Ingreso) from Colaboradores c2 where c2.nId_Colaborador = sv.nId_Colaborador)
									THEN
										DATEADD(YEAR, 1,@dFecha)
									ELSE
										sv.dUltimo_Periodo_Ganado
									END,
		sv.nUsuario_Update = @nUsuario_Update,
		sv.dDatetime_Update = dbo.function_fecha_actual()
	FROM saldos_Vacaciones_Colaboradores AS sv;
	END	

	BEGIN
	Update sv 
	SET 
		dDias_Ganados_Laborales_new = (select TOP 1 dDias_Ganados_Laborales from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador) ,
		dDias_Ganados_Laborales_has_changed = 	CASE 
													When (select TOP 1 dDias_Ganados_Laborales from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador) != dDias_Ganados_Laborales_old then 1
													else 0
												END,
		dDias_Vencidos_Laborales_new = (select TOP 1 dDias_Vencidos_Laborales from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador)  ,
		dDias_Vencidos_Laborales_has_changed = 	CASE 
													When (select TOP 1 dDias_Vencidos_Laborales from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador) != dDias_Vencidos_Laborales_old then 1
													else 0
												END,
		dDias_Generados_Laborales_new = (select TOP 1 dDias_Generados_Laborales from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador)  ,
		dDias_Generados_Laborales_has_changed = 	CASE 
													When (select TOP 1 dDias_Generados_Laborales from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador)  != dDias_Generados_Laborales_old then 1
													else 0
												END,
		
		dDias_Ganados_Calendario_new = (select TOP 1 dDias_Ganados_Calendario from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador)  ,
		dDias_Ganados_Calendario_has_changed = 	CASE 
													When (select TOP 1 dDias_Ganados_Calendario from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador) != dDias_Ganados_Calendario_old then 1
													else 0
												END,
		dDias_Vencidos_Calendario_new = (select TOP 1 dDias_Vencidos_Calendario from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador) ,
		dDias_Vencidos_Calendario_has_changed = 	CASE 
													When (select TOP 1 dDias_Vencidos_Calendario from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador) != dDias_Vencidos_Calendario_old then 1
													else 0
												END,
		dDias_Generados_Calendario_new = (select TOP 1dDias_Generados_Calendario from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador),
		dDias_Generados_Calendario_has_changed = CASE 
													When (select TOP 1 dDias_Generados_Calendario from saldos_Vacaciones_Colaboradores nuevos_saldos where nuevos_saldos.nId_Colaborador = sv.nId_Colaborador) != dDias_Generados_Calendario_old then 1
													else 0
												 END
		from @saldos_vacaciones as sv
	END	
	
	
	declare @preview_logs table(nId int, sEntidad nvarchar(max), sColumn nvarchar(max), dOldValue decimal(18,6), dNewValue decimal(18,6))
	
	Insert Into @preview_logs
	select 
		nId_Colaborador,
		'Colaborador',
		'Dias_Ganados_Laborales',
		dDias_Ganados_Laborales_old,
		dDias_Ganados_Laborales_new
	from 
		@saldos_vacaciones 
	where
		dDias_Ganados_Laborales_has_changed = 1
		
	Insert Into @preview_logs
	select 
		nId_Colaborador,
		'Colaborador',
		'Dias_Vencidos_Laborales',
		dDias_Vencidos_Laborales_old,
		dDias_Vencidos_Laborales_new
	from 
		@saldos_vacaciones 
	where
		dDias_Vencidos_Laborales_has_changed = 1

	Insert Into @preview_logs
	select 
		nId_Colaborador,
		'Colaborador',
		'Dias_Generados_Laborales',
		dDias_Generados_Laborales_old,
		dDias_Generados_Laborales_new
	from 
		@saldos_vacaciones 
	where
		dDias_Generados_Laborales_has_changed = 1
	
	Insert Into @preview_logs
	select 
		nId_Colaborador,
		'Colaborador',
		'Dias_Ganados_Calendario',
		dDias_Ganados_Calendario_old,
		dDias_Ganados_Calendario_new
	from 
		@saldos_vacaciones 
	where
		dDias_Ganados_Calendario_has_changed = 1
	
	Insert Into @preview_logs
	select 
		nId_Colaborador,
		'Colaborador',
		'Dias_Vencidos_Calendario',
		dDias_Vencidos_Calendario_old,
		dDias_Vencidos_Calendario_new
	from 
		@saldos_vacaciones 
	where
		dDias_Vencidos_Calendario_has_changed = 1
	
	Insert Into @preview_logs
	select 
		nId_Colaborador,
		'Colaborador',
		'Dias_Generados_Calendario',
		dDias_Generados_Calendario_old,
		dDias_Generados_Calendario_new
	from 
		@saldos_vacaciones 
	where
		dDias_Generados_Calendario_has_changed = 1
END
	INSERT INTO Tb_log_procedure_changes
	SELECT 'sp_calcular_vacaciones', 
	CONCAT('Los ',sColumn, ' del colaborador con id ',nId,' cambio de ',dOldValue,' a ',dNewValue),
	'@dFecha,@nUsuario_Update', 
	CONCAT(@dFecha,',',@nUsuario_Update), 
	@nUsuario_Update, dbo.function_fecha_actual(), NULL,NULL,NULL,NULL,
    -- new column to insert a json with the value of all the old values
    (
        SELECT
            dDias_Ganados_Laborales_old,
            dDias_Vencidos_Laborales_old,
            dDias_Generados_Laborales_old,
            dDias_Ganados_Calendario_old,
            dDias_Vencidos_Calendario_old,
            dDias_Generados_Calendario_old
        FROM @saldos_vacaciones
        WHERE nId_Colaborador = nId
        FOR JSON PATH
    )
	from @preview_logs