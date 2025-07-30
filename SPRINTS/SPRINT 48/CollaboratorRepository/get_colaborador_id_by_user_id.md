```sql
  
create function fn_get_colaborador_id_by_user_id(nid_user integer) returns integer  
    language plpgsql  
as  
$$  
declare  
    nid_collaborator INTEGER;  
begin  
    select c.nid_colaborador  
    into nid_collaborator  
    from usuarios u  
             join personas p ON u.nid_persona = p.nid_persona  
             join colaboradores c ON p.nid_persona = c.nid_persona  
    where u.nid_usuario = nid_user;  
    return nid_collaborator;  
end;  
$$;
```