```sql
CREATE OR REPLACE FUNCTION fn_get_all_areas()
RETURNS SETOF TEXT AS $$
BEGIN
    RETURN QUERY
    SELECT sdescripcion
    FROM areas;
END;
$$ LANGUAGE plpgsql;