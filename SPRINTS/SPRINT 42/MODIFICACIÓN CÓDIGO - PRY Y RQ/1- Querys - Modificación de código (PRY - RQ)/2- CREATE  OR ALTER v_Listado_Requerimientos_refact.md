```SQL

CREATE OR ALTER VIEW v_Listado_Requerimientos_refact

AS

SELECT

r.nId_Requerimiento,

r.sNombre AS sNombre_Requerimiento,

CONCAT(r.sPrefijo, ' - ', r.sNombre_SinPrefijo) AS sNombre_Requerimiento_ConPrefijo,

r.sPrefijo,

r.nId_Proyecto,

r.nId_Lider AS nId_Lider_Rq,

p.nId_Lider AS nLiderPrincipal,

CONCAT(pLider.sPrimer_Nombre, ' ', pLider.sApe_Paterno) AS sNombreLider,

CONCAT(pDir.sPrimer_Nombre, ' ', pDir.sApe_Paterno) AS sNombreDirector,

p.sNombre AS sNombre_Proyecto,

p.sCodigo AS sCodigo_proyecto,

CONCAT(cPrefProj.sDescripcion, ' - ', p.sNombre_Proyecto_Sin_Prefijo) as sNombre_Proyecto_ConPrefijo,

cPrefProj.sDescripcion AS sPrefijoPRY,

r.sNombre_SinPrefijo,

LTRIM(SUBSTRING(r.sCodigo, 3, LEN(r.sCodigo))) AS sCodigo_sinprefijo,

CONVERT(DATE, r.dFecha_Inicio) AS dFecha_Inicio,

(

SELECT MIN(dFecha_Registro)

FROM Tareas

WHERE nId_Proyecto = p.nId_Proyecto

AND nId_Requerimiento = r.nId_Requerimiento

) AS dFechaInicioRqReal,

CONVERT(DATE, r.dFecha_Fin) AS dFecha_Fin,

dbo.fn_get_Real_End_Date_Requirement(r.nId_Requerimiento) AS dFechaFinRqReal,

r.dFecha_Modificacion_Estado,

p_coordinador.sPersona_Nombre AS Nombre_Coordinador,

p2.sPersona_Nombre AS Nombre_Cliente,

r.nEstado AS nEstado_Requerimiento,

prmc.nEstimado AS nEstimacion,

cEstadoReq.sDescripcion AS sEstado_Requerimiento,

CASE

WHEN (

SELECT COUNT(*)

FROM justificaciones_Proyectos_Requerimientos jr

WHERE jr.nId_Entidad = r.nId_Requerimiento

AND jr.sEntidad = 'REQUERIMIENTO'

) > 0

THEN convert(bit, 1)

ELSE convert(bit, 0)

END AS bJustificacion,

convert(datetime2(7), r.dDatetime_Creador) AS dFecha_Creacion,

CASE

WHEN r.nUsuario_Creador IN (172, 666)

THEN 'SMART'

ELSE CONCAT(pCrea.sPrimer_Nombre, ' ', pCrea.sApe_Paterno)

END AS sNombre_Usuario_Creador,

cTipoServ.sDescripcion AS sTipo_Servicio_Req,

r.sCodigo

FROM

Requerimientos r

JOIN Proyectos p ON r.nId_Proyecto = p.nId_Proyecto

JOIN Colaboradores cLider ON r.nId_Lider = cLider.nId_Colaborador

JOIN Personas pLider ON pLider.nId_Persona = cLider.nId_Persona

JOIN Colaboradores cDir ON p.nId_Director = cDir.nId_Colaborador

JOIN Personas pDir ON pDir.nId_Persona = cDir.nId_Persona

JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto

JOIN Coordinadores c ON c.nId_Coordinador = cp.nId_Coordinador

JOIN Personas p_coordinador ON p_coordinador.nId_Persona = c.nId_Persona

JOIN Clientes c2 ON c.nId_Cliente = c2.nId_Cliente

JOIN Personas p2 ON c2.nId_Persona = p2.nId_Persona

JOIN pryRqMinutosCalculados prmc ON prmc.nId_Requerimiento = r.nId_Requerimiento

JOIN Configs cEstadoReq ON cEstadoReq.sCodigo = CONVERT(VARCHAR, r.nEstado)

AND cEstadoReq.sTabla = 'ESTADO_REQUERIMIENTO'

JOIN Usuarios uCrea ON uCrea.nId_Usuario = r.nUsuario_Creador

JOIN Personas pCrea ON pCrea.nId_Persona = uCrea.nId_Persona

JOIN Configs cTipoServ ON cTipoServ.sCodigo = p.sId_Tipo_Servicio

AND cTipoServ.sTabla = 'TIPO_SERVICIO'

JOIN Configs cPrefProj ON cPrefProj.sCodigo = CAST(p.sPrefijo AS VARCHAR(10))

AND cPrefProj.sTabla = 'PREFIJO_PROYECTO';