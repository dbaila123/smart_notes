```sql
create or replace function fn_get_collaborator_by_id_collaborator(p_nid_collaborator INTEGER)
returns setof colaboradores_logs as $$
begin
	return query
	SELECT cl.*
	FROM colaboradores_logs cl
	WHERE nid_colaborador = p_nid_collaborator;
end;
$$ language plpgsql;