```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_contratos_historicos(
    par_nid_contrato INT,
    par_nestado INT DEFAULT 1
)
RETURNS TABLE (
    nid_historico INT,
    nid_contrato INT,
    sjson_contrato_old TEXT,
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
        ch.nid_historico,
        ch.nid_contrato,
        ch.sjson_contrato_old,
        ch.nestado,
        ch.nusuario_creador,
        ch.ddatetime_creador,
        ch.nusuario_update,
        ch.ddatetime_update,
        ch.nusuario_delete,
        ch.ddatetime_delete
    FROM tenant0001_dbo.contratos_historicos ch
    WHERE ch.nid_contrato = par_nid_contrato
      AND ch.nestado = par_nestado;
END;
$$;
```