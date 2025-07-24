```sql
create function fn_get_user_id_by_session_token(p_session_token text) returns integer  
    language plpgsql  
as  
$$  
DECLARE  
    v_user_id integer;  
BEGIN  
    SELECT nid_usuario  
    INTO v_user_id  
    FROM sesiones  
    WHERE  
        stoken_usuario = p_session_token  
        AND nestado = 1  
    LIMIT 1;  
  
    RETURN v_user_id;  
END;  
$$;