```sql
CREATE OR REPLACE FUNCTION fn_smart_mg_services_get_document_of_client(p_nid_persona INT)
RETURNS SETOF documentos
AS $$
BEGIN
    RETURN QUERY
    SELECT d.*
    FROM documentos d
    JOIN personas p ON d.nid_persona = p.nid_persona
    WHERE p.nid_persona = p_nid_persona;
END;
$$ LANGUAGE plpgsql;