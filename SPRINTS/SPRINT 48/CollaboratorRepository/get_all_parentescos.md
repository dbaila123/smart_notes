```sql
CREATE OR REPLACE FUNCTION fn_get_all_parentescos()
RETURNS SETOF text as $$
BEGIN
    RETURN QUERY
    SELECT sdescripcion
    FROM configs
    WHERE stabla = 'PARENTESCO';
END;
$$ LANGUAGE plpgsql;