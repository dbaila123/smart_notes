```sql
create   procedure sp_smart_get_descuentos_solicitudes
(
	@nId INT,
	@numFilter int
)
 AS
 BEGIN
 	SELECT 
		s.nId_Solictud nId,
		tsm.nId_Transaccion,
		c.nId_Colaborador nId_Colaborador,
		peco.sUrl_Foto sUrl_Foto,
		p.sPersona_Nombre sNombre_Colaborador,
		(CASE
  			WHEN s.nId_Tarea IS NOT NULL
			THEN CONVERT(varchar, Tareas.dDatetime_Creador, 120)
			ELSE CONVERT(varchar, s.dDatetime_Creador, 120)
		END) dFecha_Solicitud,
		s.nId_Tipo_Solicitud nId_Tipo_Solicitud,
		ts.sDescripcion sNombre_Tipo_Solicitud,
		CONCAT(CONVERT(varchar, s.dFecha_Inicio), ' ', CONVERT(varchar, s.dHora_Inicio)) dFecha_Inicio,
		CONCAT(CONVERT(varchar, s.dFecha_Fin), ' ', CONVERT(varchar, s.dHora_Fin)) dFecha_Fin,
		SUM(s.nCantidad) nCantidad, -- Agregación para sumar cantidades
		tb_conf_modalidad.sCodigo nTipo_Modalidad,
		tb_conf_modalidad.sDescripcion sNombre_Modalidad,
		se.nId_Sede nSede,
		se.sDescripcion sNombre_Sede,
		s.nUsuario_Update nUsuario_Update,
		isnull(sh.nTotal_sh, 1) nFlag_Historico,
		(CASE
			WHEN
			  s.nUsuario_Update IS NULL THEN (
				CASE
					WHEN
					(s.nId_Tipo_Solicitud = 9 AND
					s.nEstado_Solicitud = 3) OR
					(s.nUsuario_Creador != peco.nId_Usuario) THEN 'AUTOMÁTICO'
					ELSE ''
				END
				)
			ELSE (
					SELECT TOP 1
						CONCAT(tb_person_update.sPrimer_Nombre, ' ', tb_person_update.sApe_Paterno)
					FROM Usuarios tb_user_update
					JOIN Personas tb_person_update
						ON tb_user_update.nId_Persona = tb_person_update.nId_Persona
					WHERE tb_user_update.nId_Usuario = s.nUsuario_Update
				--     )
				-- END
			)
		END) sNombre_Usuario_Update,
		(CASE
			-- extra hours
			WHEN (s.nId_Tipo_Solicitud = 9 AND s.nEstado_Solicitud = 3) THEN '[]'
			-- is created by rrhh
			WHEN (s.nUsuario_Creador != peco.nId_Usuario) THEN '[]'
			ELSE s.sAprobadores
			END
		) aprobadores,
		s.nEstado_Solicitud nEstado_Solicitud,
		cfgs.sDescripcion sEstado_Solicitud,
		isnull(s.nId_Proyecto, 0) nId_Proyecto,
		isnull(prj.sNombre, '') as sNombre_Proyecto,
		isnull(req.sNombre, '') as sNombre_Requerimiento,
		isnull(cat.sNombre, '') as sNombre_Categoria_Requerimiento,
		s.nId_Tipo_Motivo nId_Tipo_Motivo,
		tm.sDescripcion sNombre_Tipo_Motivo,
		s.sMotivo sMotivo,
		tb_conf_modalidad.sIcono sIcono_Tipo_Modalidad,
		(CASE
			WHEN peco.nId_Usuario = s.nUsuario_Creador THEN NULL
			ELSE CONCAT(creator.sPrimer_Nombre, ' ', creator.sApe_Paterno)
		END) sPersona_Nombre_Creator,
		ts.nId_Categoria nId_Categoria,
		ts.nTipo_Frecuencia,
		CONCAT(p.sPrimer_Nombre, ' ', p.sApe_Paterno) sNombre_Lider,
		cs.nId_Supervisor nId_Lider,

 		tsm.nId_Sub_Tipo_Entidad,
 		tsm.nId_Tipo_Entidad,
 		tsm.nTipo_Transaccion,
		s.dDatetime_Creador,
		s.nUsuario_Creador,
		s.nId_Supervisor,
		s.sObservacion_Lider
    FROM Transacciones_Saldo_Mins tsm
		JOIN Solicitudes s ON tsm.nId_Entidad = s.nId_Solictud and cast (s.nEstado_Solicitud as int) = 3
		JOIN Configs cfgs on cfgs.sCodigo = s.nEstado_Solicitud and cfgs.stabla = 'ESTADO_SOLICITUD'
		JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador 
		JOIN Personas p ON p.nId_Persona = c.nId_Persona 
		JOIN Tipos_Solicitudes ts on ts.nId_Tipo_Solicitud  = s.nId_Tipo_Solicitud 
		JOIN vw_user_person_collab peco ON s.nId_Colaborador = peco.nId_Colaborador
		JOIN colaboradores_supervisor cs ON cs.nId_Colaborador = s.nId_Colaborador
		JOIN Sedes_Colaboradores sc ON peco.nId_Colaborador = sc.nId_Colaborador
		JOIN Sedes se ON se.nId_Sede = sc.nId_Sede
		JOIN Configs tb_conf_modalidad ON CAST(peco.nTipo_Modalidad as VARCHAR(10)) = tb_conf_modalidad.sCodigo AND tb_conf_modalidad.sTabla = 'TIPO_MODALIDAD'
		JOIN Tipos_Motivos tm ON tm.nId_Tipo_Motivo = s.nId_Tipo_Motivo AND cs.bPrincipal = 1
		JOIN vw_user_person_collab creator ON creator.nId_Usuario = s.nUsuario_Creador
		LEFT JOIN (SELECT COUNT(*) AS nTotal_sh, nId_Solicitud FROM v_Listado_Historico_Solicitudes	GROUP BY nId_Solicitud) sh ON sh.nId_Solicitud = s.nId_Solictud
		LEFT JOIN Proyectos prj ON s.nId_Proyecto = prj.nId_Proyecto
		LEFT JOIN Tareas tareas ON s.nId_Tarea = tareas.nId_Tarea
		LEFT JOIN Requerimientos req ON tareas.nId_Requerimiento = req.nId_Requerimiento
		LEFT JOIN Categorias cat ON CAST(tareas.sId_Categoria AS INT) = cat.nId_Categoria
    WHERE
		tsm.nId_Colaborador = @numFilter and 
		ts.b_recuperable = 1 and
		tsm.nId_Cierre IS NULL
    GROUP BY 
		s.nId_Solictud, c.nId_Colaborador, p.sPersona_Nombre,
		tsm.nId_Sub_Tipo_Entidad, tsm.nId_Tipo_Entidad, tsm.nTipo_Transaccion, tsm.nId_Transaccion,
		s.nId_Tipo_Solicitud, s.dDatetime_Creador, s.nUsuario_Creador,
		s.sMotivo, s.nId_Supervisor, s.nId_Tipo_Motivo,
		s.sObservacion_Lider, s.sAprobadores, s.dFecha_Solicitud,
		ts.nId_Categoria, CONCAT(p.sPrimer_Nombre, ' ', p.sApe_Paterno), cs.nId_Supervisor,
		s.nEstado_Solicitud, se.nId_Sede, se.sDescripcion,
		ts.nTipo_Frecuencia, tb_conf_modalidad.sDescripcion, tb_conf_modalidad.sIcono,
		tm.sDescripcion, ts.sDescripcion, cfgs.sDescripcion,
		peco.sUrl_Foto, s.dFecha_Inicio, s.dHora_Inicio,
		s.dFecha_Fin, s.dHora_Fin, tb_conf_modalidad.sCodigo,
		s.nUsuario_Update, sh.nTotal_sh, s.nId_Tipo_Solicitud,
		s.nUsuario_Creador, peco.nId_Usuario, s.nId_Proyecto,
		prj.sNombre, req.sNombre, cat.sNombre,
		creator.sPrimer_Nombre, creator.sApe_Paterno, Tareas.dDatetime_Creador,
		s.nId_Tarea, ts.nTipo_Frecuencia
END


```


```sql
	create or alter procedure sp_smart_get_descuentos_solicitudes
	(
		@nId INT,
		@numFilter int
	)
	AS
	BEGIN
		SELECT 
			s.nId_Solictud nId,
			tsm.nId_Transaccion,
			c.nId_Colaborador nId_Colaborador,
			peco.sUrl_Foto sUrl_Foto,
			p.sPersona_Nombre sNombre_Colaborador,
			(CASE
				WHEN s.nId_Tarea IS NOT NULL
				THEN CONVERT(varchar, Tareas.dDatetime_Creador, 120)
				ELSE CONVERT(varchar, s.dDatetime_Creador, 120)
			END) dFecha_Solicitud,
			s.nId_Tipo_Solicitud nId_Tipo_Solicitud,
			ts.sDescripcion sNombre_Tipo_Solicitud,
			CONCAT(CONVERT(varchar, s.dFecha_Inicio), ' ', CONVERT(varchar, s.dHora_Inicio)) dFecha_Inicio,
			CONCAT(CONVERT(varchar, s.dFecha_Fin), ' ', CONVERT(varchar, s.dHora_Fin)) dFecha_Fin,
			SUM(s.nCantidad) nCantidad, -- Agregación para sumar cantidades
			tb_conf_modalidad.sCodigo nTipo_Modalidad,
			tb_conf_modalidad.sDescripcion sNombre_Modalidad,
			se.nId_Sede nSede,
			se.sDescripcion sNombre_Sede,
			s.nUsuario_Update nUsuario_Update,
			isnull(sh.nTotal_sh, 1) nFlag_Historico,
			(CASE
				WHEN
				s.nUsuario_Update IS NULL THEN (
					CASE
						WHEN
						(s.nId_Tipo_Solicitud = 9 AND
						s.nEstado_Solicitud = 3) OR
						(s.nUsuario_Creador != peco.nId_Usuario) THEN 'AUTOMÁTICO'
						ELSE ''
					END
					)
				ELSE (
						SELECT TOP 1
							CONCAT(tb_person_update.sPrimer_Nombre, ' ', tb_person_update.sApe_Paterno)
						FROM Usuarios tb_user_update
						JOIN Personas tb_person_update
							ON tb_user_update.nId_Persona = tb_person_update.nId_Persona
						WHERE tb_user_update.nId_Usuario = s.nUsuario_Update
					--     )
					-- END
				)
			END) sNombre_Usuario_Update,
			(CASE
				-- extra hours
				WHEN (s.nId_Tipo_Solicitud = 9 AND s.nEstado_Solicitud = 3) THEN '[]'
				-- is created by rrhh
				WHEN (s.nUsuario_Creador != peco.nId_Usuario) THEN '[]'
				ELSE s.sAprobadores
				END
			) aprobadores,
			s.nEstado_Solicitud nEstado_Solicitud,
			cfgs.sDescripcion sEstado_Solicitud,
			isnull(s.nId_Proyecto, 0) nId_Proyecto,
			isnull(prj.sNombre, '') as sNombre_Proyecto,
			isnull(req.sNombre, '') as sNombre_Requerimiento,
			isnull(cat.sNombre, '') as sNombre_Categoria_Requerimiento,
			s.nId_Tipo_Motivo nId_Tipo_Motivo,
			tm.sDescripcion sNombre_Tipo_Motivo,
			s.sMotivo sMotivo,
			tb_conf_modalidad.sIcono sIcono_Tipo_Modalidad,
			CONCAT(creator.sPrimer_Nombre, ' ', creator.sApe_Paterno) sPersona_Nombre_Creator,
			ts.nId_Categoria nId_Categoria,
			ts.nTipo_Frecuencia,
			CONCAT(p.sPrimer_Nombre, ' ', p.sApe_Paterno) sNombre_Lider,
			cs.nId_Supervisor nId_Lider,

			tsm.nId_Sub_Tipo_Entidad,
			tsm.nId_Tipo_Entidad,
			tsm.nTipo_Transaccion,
			s.dDatetime_Creador,
			s.nUsuario_Creador,
			s.nId_Supervisor,
			s.sObservacion_Lider
		FROM Transacciones_Saldo_Mins tsm
			JOIN Solicitudes s ON tsm.nId_Entidad = s.nId_Solictud and cast (s.nEstado_Solicitud as int) = 3
			JOIN Configs cfgs on cfgs.sCodigo = s.nEstado_Solicitud and cfgs.stabla = 'ESTADO_SOLICITUD'
			JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador 
			JOIN Personas p ON p.nId_Persona = c.nId_Persona 
			JOIN Tipos_Solicitudes ts on ts.nId_Tipo_Solicitud  = s.nId_Tipo_Solicitud 
			JOIN vw_user_person_collab peco ON s.nId_Colaborador = peco.nId_Colaborador
			JOIN colaboradores_supervisor cs ON cs.nId_Colaborador = s.nId_Colaborador
			JOIN Sedes_Colaboradores sc ON peco.nId_Colaborador = sc.nId_Colaborador
			JOIN Sedes se ON se.nId_Sede = sc.nId_Sede
			JOIN Configs tb_conf_modalidad ON CAST(peco.nTipo_Modalidad as VARCHAR(10)) = tb_conf_modalidad.sCodigo AND tb_conf_modalidad.sTabla = 'TIPO_MODALIDAD'
			JOIN Tipos_Motivos tm ON tm.nId_Tipo_Motivo = s.nId_Tipo_Motivo AND cs.bPrincipal = 1
			JOIN vw_user_person_collab creator ON creator.nId_Usuario = s.nUsuario_Creador
			LEFT JOIN (SELECT COUNT(*) AS nTotal_sh, nId_Solicitud FROM v_Listado_Historico_Solicitudes	GROUP BY nId_Solicitud) sh ON sh.nId_Solicitud = s.nId_Solictud
			LEFT JOIN Proyectos prj ON s.nId_Proyecto = prj.nId_Proyecto
			LEFT JOIN Tareas tareas ON s.nId_Tarea = tareas.nId_Tarea
			LEFT JOIN Requerimientos req ON tareas.nId_Requerimiento = req.nId_Requerimiento
			LEFT JOIN Categorias cat ON CAST(tareas.sId_Categoria AS INT) = cat.nId_Categoria
		WHERE
			tsm.nId_Colaborador = @numFilter and 
			ts.b_recuperable = 1 and
			tsm.nId_Cierre IS NULL
		GROUP BY 
			s.nId_Solictud, c.nId_Colaborador, p.sPersona_Nombre,
			tsm.nId_Sub_Tipo_Entidad, tsm.nId_Tipo_Entidad, tsm.nTipo_Transaccion, tsm.nId_Transaccion,
			s.nId_Tipo_Solicitud, s.dDatetime_Creador, s.nUsuario_Creador,
			s.sMotivo, s.nId_Supervisor, s.nId_Tipo_Motivo,
			s.sObservacion_Lider, s.sAprobadores, s.dFecha_Solicitud,
			ts.nId_Categoria, CONCAT(p.sPrimer_Nombre, ' ', p.sApe_Paterno), cs.nId_Supervisor,
			s.nEstado_Solicitud, se.nId_Sede, se.sDescripcion,
			ts.nTipo_Frecuencia, tb_conf_modalidad.sDescripcion, tb_conf_modalidad.sIcono,
			tm.sDescripcion, ts.sDescripcion, cfgs.sDescripcion,
			peco.sUrl_Foto, s.dFecha_Inicio, s.dHora_Inicio,
			s.dFecha_Fin, s.dHora_Fin, tb_conf_modalidad.sCodigo,
			s.nUsuario_Update, sh.nTotal_sh, s.nId_Tipo_Solicitud,
			s.nUsuario_Creador, peco.nId_Usuario, s.nId_Proyecto,
			prj.sNombre, req.sNombre, cat.sNombre,
			creator.sPrimer_Nombre, creator.sApe_Paterno, Tareas.dDatetime_Creador,
			s.nId_Tarea, ts.nTipo_Frecuencia
	END
	go
```