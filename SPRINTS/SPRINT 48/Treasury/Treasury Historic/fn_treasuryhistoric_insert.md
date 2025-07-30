```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_treasuryhistoric_insert(
    par_nid_historico_solicitud_tesoreria integer,
    par_nid_solicitud_tesoreria integer,
    par_nestado_solicitud integer,
    par_sobservacion text,
    par_nmonto_depositado numeric(18,2),
    par_ncodigo_moneda_reembolso text,
    par_nusuario_update integer,
    par_ddatetime_update timestamp,
    par_njson text
) RETURNS void
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO tenant0001_dbo.solicitud_tesoreria_historico(
        nid_solicitud_tesoreria,
        nestado_solicitud,
        sobservacion,
        nmonto_depositado,
        ncodigo_moneda_reembolso,
        nusuario_update,
        ddatetime_update,
        njson
    ) VALUES (
        par_nid_solicitud_tesoreria,
        par_nestado_solicitud,
        par_sobservacion,
        par_nmonto_depositado,
        par_ncodigo_moneda_reembolso,
        par_nusuario_update,
        par_ddatetime_update,
        par_njson
    );
END;
$$;
```