```sql
CREATE OR REPLACE FUNCTION fn_smart_mg_services_get_person_of_client(par_nid_cliente INTEGER)
RETURNS SETOF personas
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT p.*
    FROM clientes AS c
    JOIN personas AS p ON c.nid_persona = p.nid_persona
    WHERE c.nid_cliente = par_nid_cliente;
END;
$$;