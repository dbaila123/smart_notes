```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_treasuryinstallment_insert(
    par_nid_cuota integer,
    par_nid_solicitud_tesoreria integer,
    par_nmonto numeric(18,2),
    par_dfecha_pago date,
    par_nestado integer,
    par_nusuario_creador integer,
    par_ddatetime_creador timestamp,
    par_nusuario_update integer,
    par_ddatetime_update timestamp,
    par_nusuario_delete integer,
    par_ddatetime_delete timestamp
) RETURNS void
LANGUAGE plpgsql
AS $$
BEGIN
    INSERT INTO tenant0001_dbo.tesoreria_cuotas (
        nid_solicitud_tesoreria,
        nmonto,
        dfecha_pago,
        nestado,
        nusuario_creador,
        ddatetime_creador,
        nusuario_update,
        ddatetime_update,
        nusuario_delete,
        ddatetime_delete
    )
    VALUES (
        par_nid_solicitud_tesoreria,
        par_nmonto,
        par_dfecha_pago,
        par_nestado,
        par_nusuario_creador,
        par_ddatetime_creador,
        par_nusuario_update,
        par_ddatetime_update,
        par_nusuario_delete,
        par_ddatetime_delete
    );
END;
$$;
```