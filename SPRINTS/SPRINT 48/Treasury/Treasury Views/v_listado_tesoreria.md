```SQL
CREATE OR REPLACE VIEW tenant0001_dbo.v_listado_tesoreria
AS SELECT sol_tes.nid_solicitud_tesoreria,
    sol_tes.nid_colaborador,
    col.surl_foto,
    per.spersona_nombre AS snombre_colaborador,
    sol_tes.dfecha_efecto::timestamp without time zone AS dfecha_efecto,
    sol_tes.ddatetime_creador::timestamp without time zone AS dfecha_creacion,
    sol_tes.nmonto::numeric AS nmonto,
    sol_tes.smotivo,
    sol_tes.ntipo_moneda,
    cfg_mon.sicono AS stipo_moneda,
    sol_tes.nid_tipo_solicitud,
    cfg_type_sol.sdescripcion AS stipo_solicitud,
    cat.nid_categoria,
    ( SELECT concat(tb_person_update.sprimer_nombre, ' ', tb_person_update.sape_paterno) AS snombre_usuario_update
           FROM usuarios tb_user_update
             JOIN personas tb_person_update ON tb_user_update.nid_persona = tb_person_update.nid_persona
          WHERE tb_user_update.nid_usuario = sol_tes.nusuario_update
         LIMIT 1) AS snombre_usuario_update,
    ( SELECT h.sobservacion
           FROM solicitud_tesoreria_historico h
          WHERE h.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria
          ORDER BY h.ddatetime_update DESC
         LIMIT 1) AS sobservacion,
    sol_tes.nestado_solicitud,
    cfg_sol.sdescripcion AS sestado_solicitud,
    v_part.nid_proyecto,
    v_part.sdescripcion_proyecto AS snombre_proyecto,
    v_part.nid_requerimiento,
    v_part.sdescripcion_requerimiento AS snombre_requerimiento,
    v_part.nid_lider_pry,
    v_part.nid_director_pry,
    cuo_con.ncuotas_pagadas,
    cuo_con.nmonto_por_pagar::numeric AS nmonto_por_pagar,
    sol_tes.nmonto_depositado::numeric AS nmonto_depositado,
    sol_tes.ncodigo_moneda_reembolso::text AS ncodigo_moneda_reembolso,
    cfg_mont_reembolzado.sicono AS stipo_moneda_reembolso,
        CASE
            WHEN sol_tes.nestado_solicitud = 5 THEN ( SELECT count(*)::integer AS count
               FROM tesoreria_cuotas cuo_1
              WHERE cuo_1.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria)
            ELSE cuo_con.ntotal_cuotas
        END AS ntotal_cuotas,
    ( SELECT string_agg(f.surl_file, '*'::text) AS string_agg
           FROM files f
          WHERE f.nid_entidad = sol_tes.nid_solicitud_tesoreria AND f.sentidad = 'Solicitud_Tesoreria'::text AND f.nestado = 1 AND f.nusuario_creador = sol_tes.nusuario_creador) AS surl_files_on_creation,
        CASE
            WHEN sol_tes.nestado_solicitud = ANY (ARRAY[2, 3]) THEN ( SELECT string_agg(f.surl_file, '*'::text) AS string_agg
               FROM files f
              WHERE f.nid_entidad = sol_tes.nid_solicitud_tesoreria AND f.sentidad = 'Solicitud_Tesoreria'::text AND f.nestado = 1 AND f.nusuario_creador = sol_tes.nusuario_update)
            ELSE NULL::text
        END AS surl_files_on_refund,
    ( SELECT json_agg(json_build_object('nid_proyecto', COALESCE(p.nid_proyecto, 0), 'sdescripcion_proyecto', COALESCE(TRIM(BOTH FROM concat(ps.scodigo, ' ', ps.snombre)), ''::text), 'nid_requerimiento', COALESCE(p.nid_requerimiento, 0), 'sdescripcion_requerimiento', COALESCE(concat(rq.scodigo, ' ', rq.snombre), ''::text), 'nusuario_creador', COALESCE(p.nusuario_creador, 0), 'ddatetime_creador', p.ddatetime_creador, 'nusuario_update', COALESCE(p.nusuario_update, 0), 'susuario_update', COALESCE(per_1.spersona_nombre, ''::text), 'ddatetime_update', p.ddatetime_update, 'participantes', ( SELECT json_agg(json_build_object('nid_colaborador', sp.nid_colaborador, 'spersona_nombre', per_2.spersona_nombre, 'bescolaboradorprincipal', sp.bescolaboradorprincipal)) AS json_agg
                   FROM solicitudes_tesoreria_participantes sp
                     LEFT JOIN colaboradores clss ON clss.nid_colaborador = sp.nid_colaborador
                     LEFT JOIN personas per_2 ON per_2.nid_persona = clss.nid_persona
                  WHERE sp.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria AND (sp.nid_proyecto IS NULL AND sp.nid_requerimiento IS NULL OR sp.nid_proyecto = p.nid_proyecto AND sp.nid_requerimiento = rq.nid_requerimiento)))) AS json_agg
           FROM ( SELECT DISTINCT COALESCE(p_1.nid_proyecto, 0) AS nid_proyecto,
                    COALESCE(ps_1.snombre, ''::text) AS sdescripcion_proyecto,
                    COALESCE(p_1.nid_requerimiento, 0) AS nid_requerimiento,
                    COALESCE(rq_1.snombre, ''::text) AS sdescripcion_requerimiento,
                    COALESCE(p_1.nusuario_creador, 0) AS nusuario_creador,
                    p_1.ddatetime_creador,
                    COALESCE(p_1.nusuario_update, 0) AS nusuario_update,
                    COALESCE(per_2.spersona_nombre, ''::text) AS susuario_update,
                    p_1.ddatetime_update
                   FROM solicitudes_tesoreria_participantes p_1
                     LEFT JOIN proyectos ps_1 ON ps_1.nid_proyecto = p_1.nid_proyecto
                     LEFT JOIN requerimientos rq_1 ON rq_1.nid_requerimiento = p_1.nid_requerimiento
                     LEFT JOIN colaboradores cls_1 ON cls_1.nid_colaborador = p_1.nusuario_update
                     LEFT JOIN personas per_2 ON per_2.nid_persona = cls_1.nid_persona
                  WHERE p_1.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria) p
             LEFT JOIN proyectos ps ON ps.nid_proyecto = p.nid_proyecto
             LEFT JOIN requerimientos rq ON rq.nid_requerimiento = p.nid_requerimiento
             LEFT JOIN colaboradores cls ON cls.nid_colaborador = p.nusuario_update
             LEFT JOIN personas per_1 ON per_1.nid_persona = cls.nid_persona) AS proyectos
   FROM solicitud_tesoreria sol_tes
     LEFT JOIN tesoreria_cuotas_consolidado cuo_con ON cuo_con.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria
     LEFT JOIN tesoreria_cuotas cuo ON cuo.nid_solicitud_tesoreria = cuo_con.nid_solicitud_tesoreria
     JOIN colaboradores col ON col.nid_colaborador = sol_tes.nid_colaborador
     JOIN personas per ON per.nid_persona = col.nid_persona
     JOIN usuarios usr ON usr.nid_persona = per.nid_persona
     JOIN configs cfg_sol ON cfg_sol.stabla = 'ESTADO_SOLICITUD_TESORERIA'::text AND cfg_sol.scodigo = sol_tes.nestado_solicitud::character varying::text
     JOIN configs cfg_mon ON cfg_mon.stabla = 'TIPO_MONEDA'::text AND cfg_mon.scodigo = sol_tes.ntipo_moneda::character varying::text
     LEFT JOIN configs cfg_mont_reembolzado ON cfg_mont_reembolzado.stabla = 'TIPO_MONEDA'::text AND cfg_mont_reembolzado.scodigo = sol_tes.ncodigo_moneda_reembolso::character varying::text
     JOIN tesoreria_tipos_solicitud cfg_type_sol ON cfg_type_sol.nid_tipo_solicitud = sol_tes.nid_tipo_solicitud
     JOIN tesoreria_categorias cat ON cat.nid_categoria = cfg_type_sol.nid_categoria
     LEFT JOIN v_participantes_tesoreria_principales v_part ON v_part.nid_solicitud_tesoreria = sol_tes.nid_solicitud_tesoreria;
```