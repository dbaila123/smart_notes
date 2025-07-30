```sql
create or replace function fn_get_directores()
returns setof INTEGER as $$
begin
    return query
    SELECT c.nid_colaborador
    FROM colaboradores c
    JOIN usuarios u ON c.nid_persona = u.nid_persona
    JOIN roles_usuarios ru ON u.nid_usuario = ru.nid_usuario
    WHERE ru.nid_rol = 10;
end;
$$ language plpgsql;