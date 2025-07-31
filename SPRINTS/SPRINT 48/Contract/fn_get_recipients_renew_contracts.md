```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_recipients_renew_contracts()
 RETURNS TABLE(semail text, stenantdbname text)
 LANGUAGE plpgsql
AS $$
DECLARE
    dbname TEXT := current_database();
BEGIN
    RETURN QUERY
    SELECT DISTINCT e.semail, dbname AS stenantdbname
    FROM emails e
    JOIN usuarios u ON u.nid_persona = e.nid_persona
    JOIN roles_usuarios ru ON u.nid_usuario = ru.nid_usuario
    JOIN personas p ON u.nid_persona = p.nid_persona
    JOIN colaboradores c ON c.nid_persona = p.nid_persona
    WHERE ru.nestado = 1
      AND c.nestado_colaborador = 1
      AND ru.nid_rol IN (4, 6, 9)
      AND e.nprincipal = 1;
END;
$$;
```