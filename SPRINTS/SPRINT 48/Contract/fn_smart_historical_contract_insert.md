```SQL
CREATE OR REPLACE FUNCTION fn_smart_historical_contract_insert(
    nid_contrato INTEGER,
    sjson_contrato_old TEXT,
    nestado INTEGER,
    nusuario_creador INTEGER,
    ddatetime_creador TIMESTAMP,
    nusuario_update INTEGER,
    ddatetime_update TIMESTAMP,
    nusuario_delete INTEGER,
    ddatetime_delete TIMESTAMP
)
RETURNS INTEGER
LANGUAGE plpgsql
AS $$
DECLARE
    nid_historico INTEGER := 0;
BEGIN
    INSERT INTO contratos_historicos (
        nid_contrato,
        sjson_contrato_old,
        nestado,
        nusuario_creador,
        ddatetime_creador,
        nusuario_update,
        ddatetime_update,
        nusuario_delete,
        ddatetime_delete
    )
    VALUES (
        nid_contrato,
        COALESCE(sjson_contrato_old, ''),
        nestado,
        nusuario_creador,
        ddatetime_creador,
        nusuario_update,
        ddatetime_update,
        nusuario_delete,
        ddatetime_delete
    )
    RETURNING contratos_historicos.nid_historico INTO nid_historico;

    RETURN nid_historico;
END;
$$;
```