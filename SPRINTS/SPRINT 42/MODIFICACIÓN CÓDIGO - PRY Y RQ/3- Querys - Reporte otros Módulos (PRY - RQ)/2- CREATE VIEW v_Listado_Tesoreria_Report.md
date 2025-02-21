```SQL
CREATE VIEW [dbo].[v_Listado_Tesoreria_Report] AS

SELECT

sol_tes.nId_Solicitud_Tesoreria AS nId_Solicitud_Tesoreria,

sol_tes.nId_Colaborador AS nId_Colaborador,

per.sPersona_Nombre AS sNombre_Colaborador,

CONVERT(DATETIME2(7), sol_tes.dFecha_Efecto) AS dFecha_Efecto,

CONVERT(DATETIME2(7), sol_tes.dDatetime_Creador) AS dFecha_Creacion,

sol_tes.nMonto AS nMonto,

sol_tes.sMotivo AS sMotivo,

sol_tes.nTipo_Moneda AS nTipo_Moneda,

cfg_mon.sIcono AS sTipo_Moneda,

sol_tes.nId_Tipo_Solicitud AS nId_Tipo_Solicitud,

cfg_type_sol.sDescripcion AS sTipo_Solicitud,

cat.nId_Categoria AS nId_Categoria,

v_part.nId_Proyecto AS nId_Proyecto,

v_part.sCodigo_Pry,

--v_part.sDescripcion_Proyecto AS sNombre_Proyecto,

v_part.sNombre_Proyecto_Report AS sNombre_Proyecto,

v_part.nId_Requerimiento AS nId_Requerimiento,

v_part.sCodigo_Rq,

--v_part.sDescripcion_Requerimiento AS sNombre_Requerimiento,

v_part.sNombre_Requerimiento_Report AS sNombre_Requerimiento,

(

SELECT TOP 1

CONCAT(tb_person_update.sPrimer_Nombre, ' ', tb_person_update.sApe_Paterno)

FROM Usuarios tb_user_update

JOIN Personas tb_person_update

ON tb_user_update.nId_Persona = tb_person_update.nId_Persona

WHERE tb_user_update.nId_Usuario = sol_tes.nUsuario_Update

) AS sNombre_Usuario_Update,

(

SELECT TOP 1 sObservacion

FROM Solicitud_Tesoreria_Historico h

WHERE h.nId_Solicitud_Tesoreria = sol_tes.nId_Solicitud_Tesoreria

ORDER BY h.dDatetime_Update DESC

) AS sObservacion,

sol_tes.nEstado_Solicitud AS nEstado_Solicitud,

cfg_sol.sDescripcion AS sEstado_Solicitud,

pry.nId_Lider AS nId_Lider_Pry,

pry.nId_Director AS nId_Director_Pry,

cuo_con.nCuotas_Pagadas AS nCuotas_Pagadas,

cuo_con.nMonto_Por_Pagar AS nMonto_Pendiente,

sol_tes.nMonto_Depositado AS nMonto_Depositado,

sol_tes.nCodigo_Moneda_Reembolso AS nCodigo_Moneda_Reembolso,

cfg_mont_reembolzado.sIcono AS sTipo_Moneda_Reembolso,

CASE

WHEN sol_tes.nEstado_Solicitud = 5

THEN

(

SELECT COUNT(*)

FROM Tesoreria_Cuotas cuo

WHERE cuo.nId_Solicitud_Tesoreria = sol_tes.nId_Solicitud_Tesoreria

)

ELSE cuo_con.nTotal_Cuotas

END AS nTotal_Cuotas

FROM

Solicitud_Tesoreria AS sol_tes

LEFT JOIN Tesoreria_Cuotas_Consolidado AS cuo_con

ON cuo_con.nId_Solicitud_Tesoreria = sol_tes.nId_Solicitud_Tesoreria

LEFT JOIN Tesoreria_Cuotas AS cuo

ON cuo.nId_Solicitud_Tesoreria = cuo_con.nId_Solicitud_Tesoreria

JOIN Colaboradores col

ON col.nId_Colaborador = sol_tes.nId_Colaborador

JOIN Personas per

ON per.nId_Persona = col.nId_Persona

JOIN Usuarios usr

ON usr.nId_Persona = per.nId_Persona

JOIN Configs cfg_sol

ON cfg_sol.sTabla = 'ESTADO_SOLICITUD_TESORERIA' AND cfg_sol.sCodigo = CAST(sol_tes.nEstado_Solicitud AS VARCHAR(10))

JOIN Configs cfg_mon

ON cfg_mon.sTabla = 'TIPO_MONEDA' AND cfg_mon.sCodigo = CAST(sol_tes.nTipo_Moneda AS VARCHAR(10))

LEFT JOIN Configs cfg_mont_reembolzado

ON cfg_mont_reembolzado.sTabla = 'TIPO_MONEDA' AND cfg_mont_reembolzado.sCodigo = CAST(sol_tes.nCodigo_Moneda_Reembolso AS VARCHAR(10))

JOIN Tesoreria_Tipos_Solicitud cfg_type_sol

ON cfg_type_sol.nId_Tipo_Solicitud = sol_tes.nId_Tipo_Solicitud

JOIN Tesoreria_Categorias cat

ON cat.nId_Categoria = cfg_type_sol.nId_Categoria

LEFT JOIN Proyectos pry

ON pry.nId_Proyecto = sol_tes.nId_Proyecto AND pry.nEstado = 1

LEFT JOIN Requerimientos rq

ON rq.nId_Requerimiento = sol_tes.nId_Requerimiento AND rq.nEstado = 1

LEFT JOIN v_Participantes_Tesoreria_Principales v_part

ON v_part.nId_Solicitud_Tesoreria = sol_tes.nId_Solicitud_Tesoreria;