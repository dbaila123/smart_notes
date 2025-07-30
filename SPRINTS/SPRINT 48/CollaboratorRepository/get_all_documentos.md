```sql
CREATE OR REPLACE FUNCTION fn_get_all_documentos()
RETURNS TABLE (snumero_documento TEXT, sdescripcion TEXT) AS $$
BEGIN
    RETURN QUERY
	SELECT d.snumero_documento::TEXT, c.sdescripcion::TEXT
	FROM documentos d
	INNER JOIN configs c ON c.scodigo::integer = d.ntipo_documento
	WHERE c.stabla = 'TIPO_DOCUMENTO';
END;
$$ LANGUAGE plpgsql;