```sql
create or replace function fn_get_worker_retired(p_collaborator_id INTEGER)
returns INTEGER as $$
declare
    estado INTEGER;
begin
    SELECT c.nestado_colaborador
    INTO estado
    FROM colaboradores c
    WHERE c.nid_colaborador = p_collaborator_id;
    RETURN estado;
end;
$$ language plpgsql;