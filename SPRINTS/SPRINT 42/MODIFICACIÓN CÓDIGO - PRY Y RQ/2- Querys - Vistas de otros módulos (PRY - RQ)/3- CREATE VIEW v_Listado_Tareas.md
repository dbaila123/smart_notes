```SQL
CREATE VIEW v_Listado_Tareas as SELECT

tb_task.nId_Tarea nId,

tb_user.nId_Usuario nId_Usuario,

tb_collab.nId_Colaborador nId_Colaborador,

tb_collab.sUrl_Foto sUrl_Foto,

tb_person.sPersona_Nombre sNombre_Colaborador,

tb_task.dFecha_Registro dFecha_Registro,

tb_task.nMinutos nMinutos,

tb_task.sTipo_Actividad sCodigo_Tipo_Actividad,

tb_conf_activity.sDescripcion sNombre_Tipo_Actividad,

tb_task.nId_Proyecto sCodigo_Proyecto,

tb_project.sNombre sNombre_Proyecto,

tb_task.sId_Categoria sCodigo_Categoria,

tb_categoria.sDescripcion sNombre_Categoria,

tb_task.sTipo_Hora sCodigo_Tipo_Horas,

tb_conf_hour.sDescripcion sNombre_Tipo_Horas,

tb_task.sDetalle sDetalle,

tb_project.sId_Tipo_Servicio sTipo_Servicio,

tb_conf_service.sDescripcion sNombreTipoServicio,

tb_client.nId_Cliente nId_Cliente,

tb_client_person.sPersona_Nombre sNombre_Cliente,

tb_person_coor.sPersona_Nombre sNombre_Coordinador,

(

SELECT TOP 1 sAprobadores

FROM Solicitudes

WHERE nId_Tarea = tb_task.nId_Tarea

ORDER BY dDatetime_Update DESC

) as sAprobadores_Solicitud,

(

SELECT TOP(1) Solicitudes.dHora_Inicio

FROM Solicitudes

WHERE nId_Tarea= tb_task.nId_Tarea

AND Solicitudes.nEstado_Solicitud NOT IN (6)

ORDER BY dDatetime_Update DESC

) dHora_Inicio,

(

SELECT TOP(1) Solicitudes.dHora_Fin

FROM Solicitudes

WHERE nId_Tarea= tb_task.nId_Tarea

AND Solicitudes.nEstado_Solicitud NOT IN (6)

ORDER BY dDatetime_Update DESC

) dHora_Fin,

(

SELECT TOP(1) Solicitudes.dFecha_Fin

FROM Solicitudes

WHERE nId_Tarea= tb_task.nId_Tarea

AND Solicitudes.nEstado_Solicitud NOT IN (6)

ORDER BY dDatetime_Update DESC

) dFecha_Fin,

CONVERT (nvarchar,tb_task.nEstado_Tarea) sEstado,

tb_conf_state.sDescripcion sNombre_Estado,

(SELECT count(*) FROM Observaciones where nId_Tarea= tb_task.nId_Tarea and Observaciones.nEstado = 1) nContador_Obs,

(SELECT count(*) FROM Tareas_Historicos where nId_Tarea= tb_task.nId_Tarea and Tareas_Historicos.nEstado = 1) nContador_Historico,

convert(NVARCHAR(MAX), tb_task.nId_Requerimiento ) sCodigo_Requerimiento,

r.sNombre sNombre_Requerimiento,

rt.nId_Revision nId_Revision,

rt.nEstado_Revision Revisado,

rev.nUsuario_Creador nUsuario_Revisor,

tb_task.dDatetime_Creador dFechaCreacion,

tb_task.dDatetime_Update dFechaModificacion,

tb_task.nId_Lider_Pry,

tb_task.nId_Director_Pry,

tb_task.nId_Lider_RQ,

tb_task.nId_Lider,

tb_task.nIdDirector,

tb_task.nEstado_Pago,

(SELECT STRING_AGG(nId_Lider, ',')

FROM Proyecto_Lider

WHERE nId_Proyecto = tb_project.nId_Proyecto

AND nEstado = 1

GROUP BY nId_Proyecto) nIds_Lideres_proyecto

FROM Tareas tb_task

left join Revision_Tareas rt on rt.nId_Tarea = tb_task.nId_Tarea

left join Revisiones rev on rev.nId_Revision = rt.nId_Revision

-- left join Solicitudes tb_request on tb_request.nId_Tarea = tb_task.nId_Tarea AND tb_request.nEstado_Solicitud NOT IN (6,7)

join Colaboradores tb_collab on tb_task.nId_Colaborador = tb_collab.nId_Colaborador

join Personas tb_person on tb_collab.nId_Persona = tb_person.nId_Persona

join Usuarios tb_user on tb_person.nId_Persona = tb_user.nId_Persona

join Proyectos tb_project on tb_task.nId_Proyecto = tb_project.nId_Proyecto

join Coordinadores_Proyectos tb_c_project on tb_project.nId_Proyecto = tb_c_project.nId_Proyecto

join Coordinadores tb_coordinator on tb_c_project.nId_Coordinador = tb_coordinator.nId_Coordinador

join Personas tb_person_coor on tb_person_coor.nId_Persona = tb_coordinator.nId_Persona

join Clientes tb_client on tb_coordinator.nId_Cliente = tb_client.nId_Cliente

join Personas tb_client_person on tb_client.nId_Persona = tb_client_person.nId_Persona

join Categorias tb_categoria on tb_categoria.nId_Categoria = CONVERT (int,tb_task.sId_Categoria)

join Configs tb_conf_activity on tb_task.sTipo_Actividad = tb_conf_activity.sCodigo and tb_conf_activity.sTabla = 'TIPO_ACTIVIDAD'

join Configs tb_conf_hour on tb_task.sTipo_Hora = tb_conf_hour.sCodigo and tb_conf_hour.sTabla = 'TIPO_HORA'

join Configs tb_conf_state on tb_task.nEstado_Tarea = tb_conf_state.sCodigo and tb_conf_state.sTabla = 'ESTADO_TAREA'

join Configs tb_conf_service on tb_project.sId_Tipo_Servicio = tb_conf_service.sCodigo and tb_conf_service.sTabla = 'TIPO_SERVICIO'

join Requerimientos r on tb_task.nId_Requerimiento = r.nId_Requerimiento;