```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_contract_by_id(
    par_nid_contrato INT
)
RETURNS TABLE (
    nid_contrato INT,
    nid_colaborador INT,
    nid_cargo INT,
    dfecha_ini DATE,
    dfecha_fin DATE,
    dsalario NUMERIC,
    sobservacion TEXT,
    dfecha_cese DATE,
    smotivo_cese TEXT,
    nestado_contrato INT,
    nestado INT,
    nusuario_creador INT,
    ddatetime_creador TIMESTAMP,
    nusuario_update INT,
    ddatetime_update TIMESTAMP,
    nusuario_delete INT,
    ddatetime_delete TIMESTAMP
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        c.nid_contrato,
        c.nid_colaborador,
        c.nid_cargo,
        c.dfecha_ini,
        c.dfecha_fin,
        c.dsalario,
        c.sobservacion,
        c.dfecha_cese,
        c.smotivo_cese,
        c.nestado_contrato,
        c.nestado,
        c.nusuario_creador,
        c.ddatetime_creador,
        c.nusuario_update,
        c.ddatetime_update,
        c.nusuario_delete,
        c.ddatetime_delete
    FROM tenant0001_dbo.contratos c
    WHERE c.nid_contrato = par_nid_contrato;
END;
$$;
```