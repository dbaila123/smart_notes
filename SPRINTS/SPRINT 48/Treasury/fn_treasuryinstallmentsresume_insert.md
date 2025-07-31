```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_treasuryinstallmentsresume_insert(
    par_nid_solicitud_tesoreria integer,
    par_ntotal_cuotas integer,
    par_nmonto_por_pagar numeric(18,2),
    par_ncuotas_pagadas integer,
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
    INSERT INTO tenant0001_dbo.tesoreria_cuotas_consolidado (
        nid_solicitud_tesoreria,
        ntotal_cuotas,
        nmonto_por_pagar,
        ncuotas_pagadas,
        nestado,
        nusuario_creador,
        ddatetime_creador,
        nusuario_update,
        ddatetime_update,
        nusuario_delete,
        ddatetime_delete
    ) VALUES (
        par_nid_solicitud_tesoreria,
        par_ntotal_cuotas,
        par_nmonto_por_pagar,
        par_ncuotas_pagadas,
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