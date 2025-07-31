```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_contratos_por_renovar()
RETURNS TABLE (
    nid_contrato INTEGER,
    spersona_nombre VARCHAR,
    dfecha_fin DATE,
    nmeses INTEGER,
    ndias INTEGER,
    sfotourl VARCHAR,
    stenantdatabase TEXT
)
LANGUAGE plpgsql
AS $$
DECLARE
    dbname TEXT := current_database();
    today DATE := CURRENT_DATE;
    search_date DATE := (CURRENT_DATE + INTERVAL '1 month')::date;
    ini_date DATE := date_trunc('month', search_date)::date;
    end_date DATE := (date_trunc('month', search_date) + INTERVAL '1 month - 1 day')::date;
    contract_id INTEGER;
BEGIN
    CREATE TEMP TABLE temp_contracts_to_renew (
        nid_contrato INTEGER
    ) ON COMMIT DROP;

    INSERT INTO temp_contracts_to_renew(nid_contrato)
    SELECT c.nid_contrato
    FROM contratos c
    WHERE c.dfecha_fin BETWEEN ini_date AND end_date
      AND c.nestado_contrato IN (1, 4);

    LOOP
        SELECT t.nid_contrato INTO contract_id
        FROM temp_contracts_to_renew t
        LIMIT 1;

        EXIT WHEN NOT FOUND;

        RETURN QUERY
        SELECT
            c.nid_contrato,
            p.spersona_nombre,
            c.dfecha_fin,
            date_part('month', age(c.dfecha_fin, today))::INTEGER AS nmeses,
            (date_part('day', age(c.dfecha_fin, today)) % 30)::INTEGER AS ndias,
            c2.surl_foto,
            dbname
        FROM contratos c
        JOIN colaboradores c2 ON c.nid_colaborador = c2.nid_colaborador
        JOIN personas p ON c2.nid_persona = p.nid_persona
        WHERE c.nid_contrato = contract_id;

        UPDATE contratos
        SET nestado_contrato = 2
        WHERE nid_contrato = contract_id;

        DELETE FROM temp_contracts_to_renew WHERE nid_contrato = contract_id;
    END LOOP;

    RETURN;
END;
$$;
```