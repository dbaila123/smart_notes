```sql
CREATE OR REPLACE FUNCTION fn_get_self_update_status(p_nid_user INTEGER)
RETURNS INTEGER AS $$
DECLARE
    result INTEGER;
BEGIN
    SELECT col.bselfupdate
    INTO result
    FROM colaboradores col
    INNER JOIN usuarios us ON us.nid_persona = col.nid_persona
    WHERE us.nid_usuario = p_nid_user;

    RETURN result;
END;
$$ LANGUAGE plpgsql;