```SQL
CREATE OR REPLACE VIEW tenant0001_dbo.v_listado_tesoreria_report AS
SELECT
    sol_tes.nid_solicitud_tesoreria::integer,
    sol_tes.nid_colaborador::integer,
    per.spersona_nombre::text AS snombre_colaborador,
    sol_tes.dfecha_efecto::timestamp without time zone,
    sol_tes.ddatetime_creador::timestamp without time zone AS dfecha_creacion,
    sol_tes.nmonto::numeric,
    sol_tes.smotivo::text,
    sol_tes.ntipo_moneda::integer,
    cfg_mon.sicono::text AS stipo_moneda,
    sol_tes.nid_tipo_solicitud::integer,
    cfg_type_sol.sdescripcion::text AS stipo_solicitud,
    cat.nid_categoria::integer,
    v_part.nid_proyecto::integer,
    v_part.scodigo_pry::text,
    v_part.snombre_proyecto_report::text AS snombre_proyecto,
    v_part.nid_requerimiento::integer,
    v_part.scodigo_rq::text,
    v_part.snombre_requerimiento_report::text AS snombre_requerimiento,
    (
        SELECT concat(tb_person_update.sprimer_nombre, ' ', tb_person_update.sape_paterno)::text
        FROM usuarios tb_user_update
        JOIN personas tb_person_update 
            ON tb_user_update.nid_persona = tb_person_update.nid_persona
        WHERE tb_user_update.nid_usuario = sol_tes.nusuario_update
        LIMIT 1
    ) AS snombre_usuario_update,
    (
        SELECT h.sobservacion::text
        FROM solicitud_tesoreria_historico h
        WHERE h.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria
        ORDER BY h.ddatetime_update DESC
        LIMIT 1
    ) AS sobservacion,
    sol_tes.nestado_solicitud::integer,
    cfg_sol.sdescripcion::text AS sestado_solicitud,
    pry.nid_lider::integer AS nid_lider_pry,
    pry.nid_director::integer AS nid_director_pry,
    cuo_con.ncuotas_pagadas::integer,
    cuo_con.nmonto_por_pagar::numeric,
    sol_tes.nmonto_depositado::numeric,
    sol_tes.ncodigo_moneda_reembolso::text,
    cfg_mont_reembolzado.sicono::text AS stipo_moneda_reembolso,
    CASE
        WHEN sol_tes.nestado_solicitud = 5 THEN
            (
                SELECT count(*)::integer
                FROM tesoreria_cuotas cuo
                WHERE cuo.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria
            )
        ELSE cuo_con.ntotal_cuotas::integer
    END AS ntotal_cuotas
FROM
    solicitud_tesoreria AS sol_tes
LEFT JOIN tesoreria_cuotas_consolidado AS cuo_con
    ON cuo_con.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria
LEFT JOIN tesoreria_cuotas AS cuo
    ON cuo.nid_solicitud_tesoreria = cuo_con.nid_solicitud_tesoreria
JOIN colaboradores col
    ON col.nid_colaborador = sol_tes.nid_colaborador
JOIN personas per
    ON per.nid_persona = col.nid_persona
JOIN usuarios usr
    ON usr.nid_persona = per.nid_persona
JOIN configs cfg_sol
    ON cfg_sol.stabla = 'ESTADO_SOLICITUD_TESORERIA' 
   AND cfg_sol.scodigo = sol_tes.nestado_solicitud::varchar
JOIN configs cfg_mon
    ON cfg_mon.stabla = 'TIPO_MONEDA' 
   AND cfg_mon.scodigo = sol_tes.ntipo_moneda::varchar
LEFT JOIN configs cfg_mont_reembolzado
    ON cfg_mont_reembolzado.stabla = 'TIPO_MONEDA' 
   AND cfg_mont_reembolzado.scodigo = sol_tes.ncodigo_moneda_reembolso::varchar
JOIN tesoreria_tipos_solicitud cfg_type_sol
    ON cfg_type_sol.nid_tipo_solicitud = sol_tes.nid_tipo_solicitud
JOIN tesoreria_categorias cat
    ON cat.nid_categoria = cfg_type_sol.nid_categoria
LEFT JOIN proyectos pry
    ON pry.nid_proyecto = sol_tes.nid_proyecto 
   AND pry.nestado = 1
LEFT JOIN requerimientos rq
    ON rq.nid_requerimiento = sol_tes.nid_requerimiento 
   AND rq.nestado = 1
LEFT JOIN v_participantes_tesoreria_principales v_part
    ON v_part.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria
ORDER BY sol_tes.nid_solicitud_tesoreria ASC;
```