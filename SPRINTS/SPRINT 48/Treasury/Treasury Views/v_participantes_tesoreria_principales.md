```SQL
CREATE OR REPLACE VIEW tenant0001_dbo.v_participantes_tesoreria_principales AS
WITH cte_solicitudes AS (
    SELECT
        stp.nid_solicitud_tesoreria,
        p.nid_proyecto,
        p.scodigo AS scodigo_pry,
        p.snombre AS sdescripcion_proyecto,
        CONCAT(cprefproy.sdescripcion, ' - ', p.snombre_proyecto_sin_prefijo) AS snombre_proyecto_report,
        rq.nid_requerimiento,
        rq.scodigo AS scodigo_rq,
        rq.snombre AS sdescripcion_requerimiento,
        CONCAT(rq.sprefijo, ' - ', rq.snombre_sinprefijo) AS snombre_requerimiento_report,
        p.nid_lider AS nid_lider_pry,
        p.nid_director AS nid_director_pry,
        ROW_NUMBER() OVER (PARTITION BY stp.nid_solicitud_tesoreria ORDER BY stp.nid_solicitud_tesoreria) AS rn
    FROM
        solicitudes_tesoreria_participantes stp
    LEFT JOIN
        proyectos AS p ON stp.nid_proyecto = p.nid_proyecto
    LEFT JOIN
        requerimientos AS rq ON stp.nid_requerimiento = rq.nid_requerimiento
    LEFT JOIN
        configs cprefproy ON cprefproy.stabla = 'PREFIJO_PROYECTO' AND cprefproy.scodigo = CAST(p.sprefijo AS VARCHAR(10))
)
SELECT
    nid_solicitud_tesoreria,
    nid_proyecto,
    scodigo_pry,
    sdescripcion_proyecto,
    snombre_proyecto_report,
    nid_requerimiento,
    scodigo_rq,
    sdescripcion_requerimiento,
    snombre_requerimiento_report,
    nid_lider_pry,
    nid_director_pry
FROM
    cte_solicitudes
WHERE
    rn = 1;
```