```SQL
CREATE VIEW v_Participantes_Tesoreria_Principales AS

WITH CTE_Solicitudes AS (

SELECT

stp.nId_Solicitud_Tesoreria,

p.nId_Proyecto,

CONCAT(p.sCodigo,' ', p.sNombre) AS sDescripcion_Proyecto,

rq.nId_Requerimiento,

CONCAT(rq.sCodigo, ' ', rq.sNombre) AS sDescripcion_Requerimiento,

p.nId_Lider AS nId_Lider_Pry,

p.nId_Director AS nId_Director_Pry,

ROW_NUMBER() OVER (PARTITION BY stp.nId_Solicitud_Tesoreria ORDER BY stp.nId_Solicitud_Tesoreria) AS rn

FROM

Solicitudes_Tesoreria_Participantes stp

LEFT JOIN

Proyectos AS p ON stp.nId_Proyecto = p.nId_Proyecto

LEFT JOIN

Requerimientos AS rq ON stp.nId_Requerimiento = rq.nId_Requerimiento

)

SELECT

nId_Solicitud_Tesoreria,

nId_Proyecto,

sDescripcion_Proyecto,

nId_Requerimiento,

sDescripcion_Requerimiento,

nId_Lider_Pry,

nId_Director_Pry

FROM

CTE_Solicitudes

WHERE

rn = 1;