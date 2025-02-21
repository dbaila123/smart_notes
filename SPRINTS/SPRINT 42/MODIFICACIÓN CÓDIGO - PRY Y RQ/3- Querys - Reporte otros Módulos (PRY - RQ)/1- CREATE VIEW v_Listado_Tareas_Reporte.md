```SQL
CREATE VIEW v_Listado_Tareas_Reporte AS

SELECT

tb_task.nId_Tarea AS nId /*FILTRO*/,

tb_task.nId_Colaborador,

tb_user.nId_Usuario AS nId_Usuario /*FILTRO*/,

tb_person.sPersona_Nombre AS sNombre_Colaborador /*Filtro*/,

tb_task.dFecha_Registro AS dFecha_Registro /*Filtro*/,

tb_task.nMinutos AS nMinutos,

tb_task.dDatetime_Creador,

tb_task.nId_Proyecto AS sCodigo_Proyecto /*FILTRO*/,

--CONCAT(tb_project.sCodigo, ' ' ,tb_project.sNombre) AS sNombre_Proyecto /*FILTRO*/,

tb_project.sCodigo AS sCodigo_Pry,

CONCAT(cPrefProy.sDescripcion, ' - ', tb_project.sNombre_Proyecto_Sin_Prefijo) AS sNombre_Proyecto /*FILTRO*/,

tb_categoria.sDescripcion AS sNombre_Categoria /*FILTRO*/,

tb_task.sTipo_Hora AS sCodigo_Tipo_Horas /*Filtro*/,

tb_conf_hour.sDescripcion AS sNombre_Tipo_Horas,

tb_task.sDetalle AS sDetalle,

tb_project.sId_Tipo_Servicio AS sTipo_Servicio /*FILTRO*/,

tb_client.nId_Cliente AS nId_Cliente /*FILTRO*/,

tb_client_person.sPersona_Nombre AS sNombre_Cliente /*FILTRO*/,

tb_person_coor.sPersona_Nombre AS sNombre_Coordinador,

tb_task.nEstado_Tarea AS sEstado /*FILTRO*/,

tb_conf_state.sDescripcion AS sNombre_Estado,

tb_task.nId_Lider_Pry,

tb_task.nId_Director_Pry,

tb_task.nId_Lider_RQ,

tb_task.nId_Lider,

tb_task.nIdDirector,

tb_task.sId_Categoria AS sCodigo_Categoria /*FILTRO*/,

tb_config_pay_state.sDescripcion AS sEstado_Pago,

CONCAT(MONTH(tb_config_cierre.dFecha_Cierre), '-', YEAR(tb_config_cierre.dFecha_Cierre) ) AS dFecha_Cierre,

CONVERT(NVARCHAR(MAX), tb_task.nId_Requerimiento) AS sCodigo_Requerimiento /*Filtro*/,

--CONCAT(r.sCodigo, ' ', r.sNombre) AS sNombre_Requerimiento /*FILTRO*/,

r.sCodigo AS sCodigo_Rq,

CONCAT(r.sPrefijo, ' - ', r.sNombre_SinPrefijo) AS sNombre_Requerimiento /*FILTRO*/,

CONCAT(tb_person_lider_pry.sApe_Paterno, ' ', tb_person_lider_pry.sApe_Materno, ' ', tb_person_lider_pry.sPrimer_Nombre) AS sNombreLiderPry,

concat(per_col.sApe_Paterno, ' ', per_col.sApe_Materno, ' ', per_col.sPrimer_Nombre) sNombreLider,

YEAR(tb_task.dFecha_Registro) AS nAnio,

MONTH(tb_task.dFecha_Registro) AS nMes,

tb_conf_service.sDescripcion AS sNombreTipoServicio /*Filtro*/,

CONVERT(date, tb_task.dDatetime_Creador) AS dFechaCreacion,

-- (

-- SELECT TOP 1 CONCAT(p.sApe_Paterno, ' ', p.sApe_Materno, ' ', p.sPrimer_Nombre)

-- FROM Aprobadores a

-- JOIN Usuarios u ON u.nId_Usuario = a.nId_Aprobador

-- JOIN Personas p ON p.nId_Persona = u.nId_Persona

-- WHERE a.nId_Entidad = tb_soli.nId_Solictud AND nOrder = 1 and sEntidad ='SOLICITUD'

-- ) AS sAprobadorOne,

--

-- (

-- SELECT TOP 1 sh.nEstado

-- FROM Solicitudes_Historicos sh

-- WHERE sh.nId_Solicitud = tb_soli.nId_Solictud

-- AND sh.nUsuario_Update = (SELECT TOP 1 nId_Aprobador

-- FROM Aprobadores a

-- WHERE a.nId_Entidad = tb_soli.nId_Solictud AND nOrder = 1 and sEntidad ='SOLICITUD')

-- ) AS nEstadoOne,

--

-- (

-- SELECT TOP 1 CONCAT(p.sApe_Paterno, ' ', p.sApe_Materno, ' ', p.sPrimer_Nombre)

-- FROM Aprobadores a

-- JOIN Usuarios u ON u.nId_Usuario = a.nId_Aprobador

-- JOIN Personas p ON p.nId_Persona = u.nId_Persona

-- WHERE a.nId_Entidad = tb_soli.nId_Solictud AND nOrder = 2 and sEntidad ='SOLICITUD'

-- ) AS sAprobadorTwo,

--

-- (

-- SELECT TOP 1 sh.nEstado

-- FROM Solicitudes_Historicos sh

-- WHERE sh.nId_Solicitud = tb_soli.nId_Solictud

-- AND sh.nUsuario_Update = (SELECT TOP 1 a.nId_Aprobador

-- FROM Aprobadores a

-- WHERE a.nId_Entidad = tb_soli.nId_Solictud AND nOrder = 2 and sEntidad ='SOLICITUD')

-- ) AS nEstadoTwo

(SELECT STRING_AGG(nId_Lider, ',')

FROM Proyecto_Lider

WHERE nId_Proyecto = tb_project.nId_Proyecto

AND nEstado = 1

GROUP BY nId_Proyecto) nIds_Lideres_proyecto

FROM Tareas tb_task

LEFT JOIN Colaboradores tb_collab ON tb_task.nId_Colaborador = tb_collab.nId_Colaborador

LEFT JOIN Personas tb_person ON tb_collab.nId_Persona = tb_person.nId_Persona

LEFT JOIN Usuarios tb_user ON tb_person.nId_Persona = tb_user.nId_Persona

LEFT JOIN Proyectos tb_project ON tb_task.nId_Proyecto = tb_project.nId_Proyecto

LEFT JOIN Coordinadores_Proyectos tb_c_project ON tb_project.nId_Proyecto = tb_c_project.nId_Proyecto

LEFT JOIN Coordinadores tb_coordinator ON tb_c_project.nId_Coordinador = tb_coordinator.nId_Coordinador

LEFT JOIN Personas tb_person_coor ON tb_person_coor.nId_Persona = tb_coordinator.nId_Persona

LEFT JOIN Clientes tb_client ON tb_coordinator.nId_Cliente = tb_client.nId_Cliente

LEFT JOIN Personas tb_client_person ON tb_client.nId_Persona = tb_client_person.nId_Persona

LEFT JOIN Colaboradores tb_collab_lider ON tb_collab_lider.nId_Colaborador = tb_task.nId_Lider

LEFT JOIN Personas tb_person_lider_collab ON tb_person_lider_collab.nId_Persona = tb_collab_lider.nId_Persona

LEFT JOIN Colaboradores tb_collab_director ON tb_collab_director.nId_Colaborador = tb_task.nIdDirector

LEFT JOIN Personas tb_person_director_collab ON tb_person_director_collab.nId_Persona = tb_collab_director.nId_Persona

LEFT JOIN Colaboradores tb_lider_pry ON tb_lider_pry.nId_Colaborador = tb_task.nId_Lider_Pry

LEFT JOIN Personas tb_person_lider_pry ON tb_person_lider_pry.nId_Persona = tb_lider_pry.nId_Persona

LEFT JOIN Colaboradores tb_director_pry ON tb_director_pry.nId_Colaborador = tb_task.nId_Director_Pry

LEFT JOIN Personas tb_person_director_pry ON tb_person_director_pry.nId_Persona = tb_director_pry.nId_Persona

LEFT JOIN Categorias tb_categoria ON tb_categoria.nId_Categoria = CONVERT(int, tb_task.sId_Categoria)

LEFT JOIN Configs tb_conf_hour ON tb_task.sTipo_Hora = tb_conf_hour.sCodigo AND tb_conf_hour.sTabla = 'TIPO_HORA'

LEFT JOIN Configs tb_conf_state ON CONVERT (nvarchar(max),tb_task.nEstado_Tarea) = tb_conf_state.sCodigo AND tb_conf_state.sTabla = 'ESTADO_TAREA'

LEFT JOIN Configs tb_conf_service ON tb_project.sId_Tipo_Servicio = tb_conf_service.sCodigo AND tb_conf_service.sTabla = 'TIPO_SERVICIO'

LEFT JOIN Requerimientos r ON tb_task.nId_Requerimiento = r.nId_Requerimiento

LEFT JOIN Configs tb_config_pay_state ON CONVERT(VARCHAR(10), tb_task.nEstado_Pago) = tb_config_pay_state.sCodigo AND tb_config_pay_state.sTabla = 'ESTADO_PAGO'

LEFT JOIN Cierre_Horas tb_config_cierre ON tb_task.nId_Cierre = tb_config_cierre.nId_Cierre

LEFT JOIN Configs cPrefProy ON cPrefProy.sTabla = 'PREFIJO_PROYECTO' AND cPrefProy.sCodigo = CAST(tb_project.sPrefijo as VARCHAR(10))

  

  

-- cambio David:

left join Colaboradores_supervisor csup on tb_task.nId_Colaborador = csup.nId_Colaborador and csup.bPrincipal = 1

left join Colaboradores col_csup on col_csup.nId_Colaborador = csup.nId_Supervisor

left join Personas per_col on per_col.nId_Persona = col_csup.nId_Persona;