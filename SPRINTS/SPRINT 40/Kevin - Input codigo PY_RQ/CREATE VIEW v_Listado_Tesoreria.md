```SQL
CREATE OR ALTER VIEW v_Listado_Tesoreria AS

SELECT

sol_tes.nId_Solicitud_Tesoreria AS nId_Solicitud_Tesoreria,

sol_tes.nId_Colaborador AS nId_Colaborador,

col.sUrl_Foto AS sUrl_Foto,

per.sPersona_Nombre AS sNombre_Colaborador, --sp applies filter

CONVERT(DATETIME2(7), sol_tes.dFecha_Efecto) AS dFecha_Efecto, --sp applies filter

CONVERT(DATETIME2(7), sol_tes.dDatetime_Creador) AS dFecha_Creacion, --sp applies filter

sol_tes.nMonto AS nMonto,

sol_tes.sMotivo AS sMotivo,

sol_tes.nTipo_Moneda AS nTipo_Moneda,

cfg_mon.sIcono AS sTipo_Moneda,

sol_tes.nId_Tipo_Solicitud AS nId_Tipo_Solicitud, --sp applies filter

cfg_type_sol.sDescripcion AS sTipo_Solicitud,

cat.nId_Categoria AS nId_Categoria, -- --sp applies filter

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

sol_tes.nEstado_Solicitud AS nEstado_Solicitud, --sp applies filter??

cfg_sol.sDescripcion AS sEstado_Solicitud,

v_part.nId_Proyecto,

v_part.sDescripcion_Proyecto AS sNombre_Proyecto,

v_part.nId_Requerimiento,

v_part.sDescripcion_Requerimiento AS sNombre_Requerimiento,

v_part.nId_Lider_Pry,

v_part.nId_Director_Pry,

cuo_con.nCuotas_Pagadas AS nCuotas_Pagadas,

cuo_con.nMonto_Por_Pagar AS nMonto_Pendiente,

sol_tes.nMonto_Depositado AS nMonto_Depositado,--Nuevo monto depositado

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

END AS nTotal_Cuotas,

(

SELECT STRING_AGG(f.sUrl_File, '*')

FROM Files f

WHERE f.nId_Entidad = sol_tes.nId_Solicitud_Tesoreria AND

f.sEntidad = 'Solicitud_Tesoreria' AND

f.nEstado = 1 AND f.nUsuario_Creador = sol_tes.nUsuario_Creador

) AS sUrl_Files_On_Creation,

CASE

WHEN sol_tes.nEstado_Solicitud = 3 OR sol_tes.nEstado_Solicitud = 2

THEN

(

SELECT STRING_AGG(f.sUrl_File, '*')

FROM Files f

WHERE f.nId_Entidad = sol_tes.nId_Solicitud_Tesoreria AND

f.sEntidad = 'Solicitud_Tesoreria' AND

f.nEstado = 1 AND

f.nUsuario_Creador = sol_tes.nUsuario_Update

)

ELSE NULL

END AS sUrl_Files_On_Refund,

(

SELECT

ISNULL(p.nId_Proyecto, 0) AS nId_Proyecto,

ISNULL(TRIM(CONCAT(ps.sCodigo, ' ', ps.sNombre)), '') AS sDescripcion_Proyecto,

ISNULL(p.nId_Requerimiento, 0) AS nId_Requerimiento,

ISNULL(CONCAT(rq.sCodigo, ' ', rq.sNombre), '') AS sDescripcion_Requerimiento,

ISNULL(p.nUsuario_Creador, 0) AS nUsuario_Creador,

p.dDatetime_Creador,

ISNULL(p.nUsuario_Update, 0) AS nUsuario_Update,

ISNULL(per.sPersona_Nombre, '') AS sUsuario_Update,

p.dDatetime_Update,

JSON_QUERY(

(

SELECT

sp.nId_Colaborador AS nId_Colaborador,

per.sPersona_Nombre,

sp.bEsColaboradorPrincipal AS bEsColaboradorPrincipal

FROM

Solicitudes_Tesoreria_Participantes sp

left join Colaboradores clss on clss.nId_Colaborador = sp.nId_Colaborador

left join Personas per on per.nId_Persona = clss.nId_Persona

WHERE

sp.nId_Solicitud_Tesoreria = sol_tes.nId_Solicitud_Tesoreria

AND

(

(sp.nId_Proyecto IS NULL AND sp.nId_Requerimiento IS NULL)

OR

(sp.nId_Proyecto = p.nId_Proyecto AND sp.nId_Requerimiento = rq.nId_Requerimiento)

)

FOR JSON PATH

)

) AS participantes

FROM

(

SELECT DISTINCT

ISNULL(p.nId_Proyecto, 0) AS nId_Proyecto,

ISNULL(ps.sNombre, '') AS sDescripcion_Proyecto,

ISNULL(p.nId_Requerimiento, 0) AS nId_Requerimiento,

ISNULL(rq.sNombre, '') AS sDescripcion_Requerimiento,

ISNULL(p.nUsuario_Creador, 0) AS nUsuario_Creador,

p.dDatetime_Creador,

ISNULL(p.nUsuario_Update, 0) AS nUsuario_Update,

ISNULL(per.sPersona_Nombre, '') AS sUsuario_Update,

p.dDatetime_Update

FROM

Solicitudes_Tesoreria_Participantes p

left join Proyectos ps on ps.nId_Proyecto = p.nId_Proyecto

left join Requerimientos rq on rq.nId_Requerimiento = p.nId_Requerimiento

left join Colaboradores cls on cls.nId_Colaborador = p.nUsuario_Update

left join Personas per on per.nId_Persona = cls.nId_Colaborador

WHERE

  

p.nId_Solicitud_Tesoreria = sol_tes.nId_Solicitud_Tesoreria

) p

left join Proyectos ps on ps.nId_Proyecto = p.nId_Proyecto

left join Requerimientos rq on rq.nId_Requerimiento = p.nId_Requerimiento

left join Colaboradores cls on cls.nId_Colaborador = p.nUsuario_Update

left join Personas per on per.nId_Persona = cls.nId_Colaborador

FOR JSON PATH

  

) AS Proyectos

  

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

LEFT JOIN v_Participantes_Tesoreria_Principales v_part

ON v_part.nId_Solicitud_Tesoreria = sol_tes.nId_Solicitud_Tesoreria;