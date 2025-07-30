```sql
CREATE OR REPLACE FUNCTION fn_get_all_tipo_documentos()
RETURNS SETOF TEXT AS $$
BEGIN
    RETURN QUERY
	SELECT sdescripcion
	FROM configs
	WHERE stabla = 'TIPO_DOCUMENTO';
END;
$$ LANGUAGE plpgsql;