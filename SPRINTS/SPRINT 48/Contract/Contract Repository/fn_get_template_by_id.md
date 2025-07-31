```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_template_by_id(
    par_nid_template INT
)
RETURNS TABLE (
    nid_template INT,
    sid_drivefile TEXT,
    stipo_contratacion TEXT,
    nestado INT,
    nempresa_usuaria INT,
    nusuario_creador INT,
    ddatetime_creador TIMESTAMP,
    nusuario_update INT,
    ddatetime_update TIMESTAMP
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        ct.nid_template,
        ct.sid_drivefile,
        ct.stipo_contratacion,
        ct.nestado,
        ct.nempresa_usuaria,
        ct.nusuario_creador,
        ct.ddatetime_creador,
        ct.nusuario_update,
        ct.ddatetime_update
    FROM tenant0001_dbo.contract_templates ct
    WHERE ct.nid_template = par_nid_template;
END;
$$;
```