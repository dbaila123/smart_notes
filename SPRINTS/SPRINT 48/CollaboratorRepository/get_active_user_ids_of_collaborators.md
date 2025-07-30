```sql
create or replace function fn_get_active_user_ids_of_collaborators()
returns table(nid_usuario INTEGER) as $$
begin
	return query
	select u.nid_usuario
	from colaboradores c
	join usuarios u on c.nid_persona = u.nid_persona
	where c.nestado_colaborador = 1 and u.nestado = 1;
end;
$$ language plpgsql;