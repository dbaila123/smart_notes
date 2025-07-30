```sql
CREATE OR REPLACE FUNCTION fn_get_state_ceased_collaborator_by_user_id(p_niduser INTEGER)
RETURNS INTEGER AS $$
DECLARE
    estado INTEGER;
BEGIN
    SELECT c.nestado_colaborador
    INTO estado
    FROM colaboradores c 
    JOIN usuarios u ON c.nid_persona = u.nid_persona 
    WHERE u.nid_usuario = p_niduser;
    RETURN estado;
END;
$$ LANGUAGE plpgsql;