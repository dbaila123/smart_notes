```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_treasury_insert(
    par_nid_colaborador integer,
    par_nid_tipo_solicitud integer,
    par_nmonto numeric(18,2),
    par_smotivo text,
    par_saprobadores text,
    par_dfecha_efecto timestamp without time zone,
    par_ntipo_moneda integer,
    par_nestado_solicitud integer,
    par_nestado integer,
    par_nusuario_creador integer,
    par_ddatetime_creador timestamp without time zone,
    par_nid_proyecto integer DEFAULT NULL,
    par_nid_requerimiento integer DEFAULT NULL,
    par_nusuario_update integer DEFAULT NULL,
    par_ddatetime_update timestamp without time zone DEFAULT NULL,
    par_nusuario_delete integer DEFAULT NULL,
    par_ddatetime_delete timestamp without time zone DEFAULT NULL,
    par_nid_solicitud_tesoreria integer DEFAULT NULL
) RETURNS integer
LANGUAGE plpgsql
AS $$
DECLARE
    new_id integer;
BEGIN
    INSERT INTO solicitud_tesoreria (
        nid_colaborador,
        nid_tipo_solicitud,
        nid_proyecto,
        nid_requerimiento,
        nmonto,
        smotivo,
        saprobadores,
        dfecha_efecto,
        ntipo_moneda,
        nestado_solicitud,
        nestado,
        nusuario_creador,
        ddatetime_creador,
        nusuario_update,
        ddatetime_update,
        nusuario_delete,
        ddatetime_delete
    )
    VALUES (
        par_nid_colaborador,
        par_nid_tipo_solicitud,
        CASE WHEN par_nid_proyecto = 0 THEN NULL ELSE par_nid_proyecto END,
        CASE WHEN par_nid_requerimiento = 0 THEN NULL ELSE par_nid_requerimiento END,
        par_nmonto,
        par_smotivo,
        par_saprobadores,
        par_dfecha_efecto,
        par_ntipo_moneda,
        par_nestado_solicitud,
        par_nestado,
        par_nusuario_creador,
        par_ddatetime_creador,
        par_nusuario_update,
        par_ddatetime_update,
        par_nusuario_delete,
        par_ddatetime_delete
    )
    RETURNING nid_solicitud_tesoreria INTO new_id;

    RETURN new_id;
END;
$$;
```