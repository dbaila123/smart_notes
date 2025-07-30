```sql
create or replace function fn_get_active_collaborator_ids()
returns table(nid_colaborador INTEGER) as $$
begin
	return query
	select c.nid_colaborador 
	from colaboradores c
	where c.nestado_colaborador = 1;
end;
$$ language plpgsql;