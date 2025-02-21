```SQL
CREATE VIEW v_Participantes_Tesoreria_Principales AS

WITH CTE_Solicitudes AS (

SELECT

stp.nId_Solicitud_Tesoreria,

p.nId_Proyecto,

--CONCAT(p.sCodigo,' ', p.sNombre) AS sDescripcion_Proyecto,

p.sCodigo AS sCodigo_Pry,

p.sNombre AS sDescripcion_Proyecto,

CONCAT(cPrefProy.sDescripcion, ' - ', p.sNombre_Proyecto_Sin_Prefijo) AS sNombre_Proyecto_Report,

rq.nId_Requerimiento,

--CONCAT(rq.sCodigo, ' ', rq.sNombre) AS sDescripcion_Requerimiento,

rq.sCodigo AS sCodigo_Rq,

rq.sNombre AS sDescripcion_Requerimiento,

CONCAT(rq.sPrefijo, ' - ', rq.sNombre_SinPrefijo) AS sNombre_Requerimiento_Report,

p.nId_Lider AS nId_Lider_Pry,

p.nId_Director AS nId_Director_Pry,

ROW_NUMBER() OVER (PARTITION BY stp.nId_Solicitud_Tesoreria ORDER BY stp.nId_Solicitud_Tesoreria) AS rn

FROM

Solicitudes_Tesoreria_Participantes stp

LEFT JOIN

Proyectos AS p ON stp.nId_Proyecto = p.nId_Proyecto

LEFT JOIN

Requerimientos AS rq ON stp.nId_Requerimiento = rq.nId_Requerimiento

LEFT JOIN

Configs cPrefProy ON cPrefProy.sTabla = 'PREFIJO_PROYECTO' AND cPrefProy.sCodigo = CAST(p.sPrefijo as VARCHAR(10))

)

SELECT

nId_Solicitud_Tesoreria,

nId_Proyecto,

sCodigo_Pry,

sDescripcion_Proyecto,

sNombre_Proyecto_Report,

nId_Requerimiento,

sCodigo_Rq,

sDescripcion_Requerimiento,

sNombre_Requerimiento_Report,

nId_Lider_Pry,

nId_Director_Pry

FROM

CTE_Solicitudes

WHERE

rn = 1;