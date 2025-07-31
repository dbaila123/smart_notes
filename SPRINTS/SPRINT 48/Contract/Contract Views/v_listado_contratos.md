```SQL
CREATE OR REPLACE VIEW tenant0001_dbo.v_listado_contratos
AS SELECT c.nid_contrato AS nid,
    p.spersona_nombre AS snombre_colaborador,
    c.dfecha_ini,
    c.dfecha_fin,
    co.ntipo_modalidad,
    cfg_modalidad.sdescripcion AS stipo_modalidad,
    sc.nid_sede,
    s.sdescripcion AS snombre_sede,
    c.nid_cargo,
    cg.sdescripcion AS snombre_cargo,
    eu.nid_empresa,
    pe.spersona_nombre AS snombre_empresa,
    co.dfec_ingreso::date AS dfec_ingreso,
    c.nestado_contrato AS nestado,
    cfg_contrato.sdescripcion AS snombre_estado,
    co.surl_foto,
    co.nestado_colaborador,
    cfg_colab.sdescripcion AS sestado_colaborador,
    ( SELECT f.surl_file
           FROM files f
          WHERE f.nid_entidad = c.nid_contrato AND f.sentidad = 'Contrato'::text
          ORDER BY f.surl_file
         LIMIT 1) AS surl_file,
    ( SELECT count(*) > 0
           FROM contratos_historicos ch
          WHERE ch.nid_contrato = c.nid_contrato) AS bhistorico,
    ( SELECT json_agg(ch.sjson_contrato_old)::text AS json_agg
           FROM contratos_historicos ch
          WHERE ch.nid_contrato = c.nid_contrato) AS sjson_contrato_old
   FROM contratos c
     JOIN colaboradores co ON c.nid_colaborador = co.nid_colaborador
     JOIN personas p ON p.nid_persona = co.nid_persona
     JOIN configs cfg_modalidad ON cfg_modalidad.stabla = 'TIPO_MODALIDAD'::text AND cfg_modalidad.scodigo = co.ntipo_modalidad::text
     JOIN configs cfg_colab ON cfg_colab.stabla = 'ESTADO_COLABORADOR'::text AND cfg_colab.scodigo = co.nestado_colaborador::text
     JOIN sedes_colaboradores sc ON sc.nid_colaborador = co.nid_colaborador
     JOIN sedes s ON s.nid_sede = sc.nid_sede
     JOIN cargos_colaboradores cc ON cc.nid_colaborador = co.nid_colaborador
     JOIN cargos cg ON cg.nid_cargo = cc.nid_cargo
     JOIN empresas_usuarias eu ON eu.nid_empresa = co.nid_empresa
     JOIN personas pe ON pe.nid_persona = eu.nid_persona
     JOIN configs cfg_contrato ON cfg_contrato.stabla = 'ESTADO_CONTRATO'::text AND cfg_contrato.scodigo = c.nestado_contrato::text
  WHERE c.nestado = 1 AND cc.nestado = 1;
```