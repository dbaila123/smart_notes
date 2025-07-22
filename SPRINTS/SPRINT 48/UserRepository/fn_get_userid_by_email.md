```sql
create function fn_get_userid_by_email(p_email text, p_state integer) returns integer  
    language plpgsql  
as  
$$  
DECLARE  
    var_nid_usuario integer;  
BEGIN  
    SELECT        nid_usuario  
    INTO var_nid_usuario  
    FROM usuarios  
    WHERE semail = p_email  
      AND nestado = p_state;  
  
    RETURN var_nid_usuario;  
end;  
$$;```