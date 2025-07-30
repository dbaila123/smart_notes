```sql
CREATE OR REPLACE FUNCTION fn_get_all_roles()
RETURNS SETOF TEXT AS $$
BEGIN
    RETURN QUERY
    SELECT sdescripcion
	FROM roles;
END;
$$ LANGUAGE plpgsql;