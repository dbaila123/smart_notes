```sql
CREATE OR REPLACE FUNCTION fn_get_all_cargos()
RETURNS SETOF TEXT AS $$
BEGIN
    RETURN QUERY
    SELECT snombre
    FROM cargos;
END;
$$ LANGUAGE plpgsql;