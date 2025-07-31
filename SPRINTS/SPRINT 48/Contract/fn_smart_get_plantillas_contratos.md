```SQL
CREATE OR REPLACE FUNCTION fn_smart_get_plantillas_contratos()
RETURNS TABLE(
    nid_template integer,
    sid_drivefile text,
    stipo_contratacion text,
    snombre_empresa_usuaria text,
    snombre_tipo_contrataciona text,
    nempresa_usuaria integer,
    nusuario_creador integer,
    ddatetime_creador timestamp,
    nusuario_update integer,
    ddatetime_update timestamp,
    total bigint
) 
LANGUAGE plpgsql
AS $$
DECLARE
    v_total bigint;
BEGIN
    SELECT COUNT(*) INTO v_total FROM tenant0001_dbo.v_listado_plantillas_contratos;
    
    RETURN QUERY 
    SELECT 
        v.nid_template,
        v.sid_drivefile,
        v.stipo_contratacion,
        v.snombre_empresa_usuaria,
        v.snombre_tipo_contrataciona,
        v.nempresa_usuaria,
        v.nusuario_creador,
        v.ddatetime_creador,
        v.nusuario_update,
        v.ddatetime_update,
        v_total AS total
    FROM tenant0001_dbo.v_listado_plantillas_contratos v;
END;
$$;
```