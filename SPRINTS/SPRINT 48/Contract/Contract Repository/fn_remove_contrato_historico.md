```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_remove_contrato_historico(
    par_nid_historico INT,
    par_nid_contrato INT
)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
DECLARE
    rows_affected INT;
BEGIN
    DELETE FROM tenant0001_dbo.contratos_historicos
    WHERE nid_contrato = par_nid_contrato
      AND nid_historico = par_nid_historico;

    GET DIAGNOSTICS rows_affected = ROW_COUNT;

    RETURN rows_affected > 0;
END;
$$;
```