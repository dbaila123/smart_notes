```sql
create or replace function fn_get_by_id_coll_async(p_nid_colaborador INTEGER)
returns setof personas as $$
begin
	return query
	SELECT p.* FROM colaboradores c
	JOIN personas p ON c.nid_persona = p.nid_persona
	WHERE c.nid_colaborador = p_nid_colaborador;
end;
$$ language plpgsql;