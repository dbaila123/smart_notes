```sql
create or replace function fn_get_person_by_id(nid_collaborator INTEGER)
returns setof personas as $$
begin
	RETURN query
	SELECT p.* FROM personas p
	JOIN colaboradores c on p.nid_persona = c.nid_persona
	WHERE c.nid_colaborador = nid_collaborator;
end;
$$ language plpgsql;