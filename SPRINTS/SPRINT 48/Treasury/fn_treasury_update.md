```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_treasury_update(
    par_nid_solicitud_tesoreria integer,
    par_nid_colaborador integer DEFAULT NULL,
    par_nid_tipo_solicitud integer DEFAULT NULL,
    par_nid_proyecto integer DEFAULT NULL,
    par_nid_requerimiento integer DEFAULT NULL,
    par_nmonto numeric DEFAULT NULL,
    par_smotivo text DEFAULT NULL,
    par_saprobadores text DEFAULT NULL,
    par_dfecha_efecto date DEFAULT NULL,
    par_ntipo_moneda integer DEFAULT NULL,
    par_nmonto_depositado numeric DEFAULT NULL,
    par_ncodigo_moneda_reembolso varchar(1) DEFAULT NULL,
    par_nestado_solicitud integer DEFAULT NULL,
    par_nestado integer DEFAULT NULL,
    par_nusuario_creador integer DEFAULT NULL,
    par_ddatetime_creador timestamp DEFAULT NULL,
    par_nusuario_update integer DEFAULT NULL,
    par_ddatetime_update timestamp DEFAULT NULL,
    par_nusuario_delete integer DEFAULT NULL,
    par_ddatetime_delete timestamp DEFAULT NULL
)
RETURNS integer
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE tenant0001_dbo.solicitud_tesoreria
    SET
        nid_colaborador = COALESCE(par_nid_colaborador, nid_colaborador),
        nid_tipo_solicitud = COALESCE(par_nid_tipo_solicitud, nid_tipo_solicitud),
        nid_proyecto = COALESCE(par_nid_proyecto, nid_proyecto),
        nid_requerimiento = COALESCE(par_nid_requerimiento, nid_requerimiento),
        nmonto = COALESCE(par_nmonto, nmonto),
        smotivo = COALESCE(par_smotivo, smotivo),
        saprobadores = COALESCE(par_saprobadores, saprobadores),
        dfecha_efecto = COALESCE(par_dfecha_efecto, dfecha_efecto),
        ntipo_moneda = COALESCE(par_ntipo_moneda, ntipo_moneda),
        nmonto_depositado = COALESCE(par_nmonto_depositado, nmonto_depositado),
        ncodigo_moneda_reembolso = COALESCE(par_ncodigo_moneda_reembolso, ncodigo_moneda_reembolso),
        nestado_solicitud = COALESCE(par_nestado_solicitud, nestado_solicitud),
        nestado = COALESCE(par_nestado, nestado),
        nusuario_creador = COALESCE(par_nusuario_creador, nusuario_creador),
        ddatetime_creador = COALESCE(par_ddatetime_creador, ddatetime_creador),
        nusuario_update = COALESCE(par_nusuario_update, nusuario_update),
        ddatetime_update = COALESCE(par_ddatetime_update, ddatetime_update),
        nusuario_delete = COALESCE(par_nusuario_delete, nusuario_delete),
        ddatetime_delete = COALESCE(par_ddatetime_delete, ddatetime_delete)
    WHERE nid_solicitud_tesoreria = par_nid_solicitud_tesoreria;

    RETURN par_nid_solicitud_tesoreria;
END;
$$;
```