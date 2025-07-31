```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_contract_template_insert(
    sid_drivefile text,
    stipocontratacion text,
    nempresa_usuaria integer,
    nusuario_creador integer,
    ddatetime_creador timestamp with time zone,
    nusuario_update integer,
    ddatetime_update timestamp with time zone,
    nestado integer
)
RETURNS integer
LANGUAGE plpgsql
AS $$
DECLARE
    nid_template integer := 0;
BEGIN
    INSERT INTO contract_templates AS ct (
        sid_drivefile,
        stipo_contratacion,
        nempresa_usuaria,
        nestado,
        nusuario_creador,
        ddatetime_creador,
        nusuario_update,
        ddatetime_update
    )
    VALUES (
        sid_drivefile,
        stipocontratacion,
        nempresa_usuaria,
        nestado,
        nusuario_creador,
        ddatetime_creador,
        nusuario_update,
        ddatetime_update
    )
    RETURNING ct.nid_template INTO nid_template;

    RETURN nid_template;
END;
$$;
```