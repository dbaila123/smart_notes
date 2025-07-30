```sql
create or replace function fn_get_by_user_id(nid_user INTEGER)
returns setof personas as $$
begin
	return query
	SELECT p.* FROM usuarios u
	JOIN personas p ON u.nid_persona = p.nid_persona
	WHERE u.nid_usuario = nid_user;
end;
$$ language plpgsql;