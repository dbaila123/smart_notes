```SQL
CREATE OR REPLACE VIEW tenant0001_dbo.v_listado_plantillas_contratos
AS SELECT ct.nid_template,
    ct.sid_drivefile,
    ct.stipo_contratacion,
    p.spersona_nombre AS snombre_empresa_usuaria,
    c.sdescripcion AS snombre_tipo_contrataciona,
    ct.nempresa_usuaria,
    ct.nusuario_creador,
    ct.ddatetime_creador,
    ct.nusuario_update,
    ct.ddatetime_update
   FROM contract_templates ct
     LEFT JOIN empresas_usuarias eu ON ct.nempresa_usuaria = eu.nid_empresa
     LEFT JOIN personas p ON eu.nid_persona = p.nid_persona
     LEFT JOIN configs c ON lower(c.scodigo) = lower(ct.stipo_contratacion) AND lower(c.stabla) = lower('TIPO_CONTRATACION'::text) AND c.nestado = 1;
```