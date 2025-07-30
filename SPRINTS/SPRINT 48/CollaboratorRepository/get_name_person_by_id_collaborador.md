```sql
create or replace function fn_get_name_person_by_id_collaborador(p_nid_colaborador INTEGER)
returns text as $$
declare
    full_name TEXT;
begin
    SELECT p.sprimer_nombre || ' ' || p.sape_paterno
    INTO full_name
    FROM colaboradores c
    JOIN personas p ON c.nid_persona = p.nid_persona
    WHERE c.nid_colaborador = p_nid_colaborador;
    return full_name;
end;
$$ language plpgsql;