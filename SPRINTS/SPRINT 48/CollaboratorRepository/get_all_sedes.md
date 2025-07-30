```sql
CREATE OR REPLACE FUNCTION fn_get_all_sedes()
RETURNS SETOF TEXT AS $$
BEGIN
    RETURN QUERY
    SELECT sdescripcion
	FROM sedes
	WHERE sentity like '%EMPRESA%' and nestado = 1;
END;
$$ LANGUAGE plpgsql;