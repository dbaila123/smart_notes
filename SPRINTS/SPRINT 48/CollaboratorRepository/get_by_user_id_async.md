```sql
create or replace function fn_get_by_user_id_async(nid_user INTEGER)
returns setof colaboradores as $$
begin
	return query
	SELECT c.* FROM usuarios u
	JOIN personas p ON u.nid_persona = p.nid_persona
	JOIN colaboradores c ON p.nid_persona = c.nid_persona
	WHERE u.nid_usuario = nid_user;
end;
$$ language plpgsql;