```sql
create or replace function fn_get_state_retired_collaborator_by_user_id(p_niduser INTEGER)
returns INTEGER as $$
declare
    estado INTEGER;
begin
    SELECT c.nestado
    INTO estado
    FROM colaboradores c 
    JOIN usuarios u ON c.nid_persona = u.nid_persona 
    WHERE u.nid_usuario = p_niduser;
    return estado;
end;
$$ language plpgsql;