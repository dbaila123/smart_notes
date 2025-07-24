```sql
create function fn_get_rol_id_by_user_id(p_nid_user integer, p_state integer) returns integer  
    language plpgsql  
as  
$$  
DECLARE  
    var_nid_rol integer;  
BEGIN  
    SELECT        ru.nid_rol  
    INTO var_nid_rol  
    FROM Roles_Usuarios ru  
    WHERE ru.nid_usuario = p_nid_user  
        and ru.nestado = p_state;  
  
    return var_nid_rol;  
end;  
$$;
```