```sql
CREATE OR REPLACE FUNCTION update_state_retired_collaborator(niduser INTEGER, stateretired INTEGER)
RETURNS VOID AS $$
BEGIN
    UPDATE colaboradores
    SET nestado = stateretired
    FROM usuarios u
    WHERE colaboradores.nid_persona = u.nid_persona
      AND u.nid_usuario = niduser;
END;
$$ LANGUAGE plpgsql;