```sql
create or replace function fn_get_collaborador_by_id(nid_collaborator INTEGER)
returns setof colaboradores as $$
begin
    return query
    select c.*
    from colaboradores c
    where c.nid_colaborador = nid_collaborator;
end;
$$ language plpgsql;