```sql
CREATE OR REPLACE FUNCTION fn_smart_mg_services_client_name_exists(par_snombrecliente VARCHAR)
RETURNS INTEGER
LANGUAGE plpgsql
AS $$
DECLARE
    resultado INTEGER;
BEGIN
    SELECT CASE
               WHEN EXISTS (
                   SELECT 1
                   FROM personas AS p
                   INNER JOIN clientes AS c
                       ON p.nid_persona = c.nid_persona
                   WHERE REGEXP_REPLACE(LOWER(TRIM(p.spersona_nombre)), '\s+', ' ', 'g') =
                         REGEXP_REPLACE(LOWER(TRIM(par_snombrecliente)), '\s+', ' ', 'g')
               ) THEN 1
               ELSE 0
           END
    INTO resultado;

    RETURN resultado;
END;
$$;