```sql 
create function fn_delete_session_by_user(p_nid_user integer) returns integer  
    language plpgsql  
as  
$$  
DECLARE  
    deleted_rows integer;  
BEGIN  
    DELETE    FROM sesiones  
    WHERE nid_usuario = p_nid_user;  
  
    GET DIAGNOSTICS deleted_rows = ROW_COUNT;  
    RETURN deleted_rows;  
END;  
$$;
```