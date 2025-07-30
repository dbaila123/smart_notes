```sql
CREATE OR REPLACE FUNCTION fn_update_state_ceased_collaborator(niduser INTEGER, stateceased INTEGER)
RETURNS VOID AS $$
BEGIN
    UPDATE colaboradores
    SET nestado_colaborador = stateceased
    FROM colaboradores c
    JOIN usuarios u ON c.nid_persona = u.nid_persona
    WHERE u.nid_usuario = niduser;
END;
$$ LANGUAGE plpgsql;