```sql
create function fn_get_userid_by_email(p_email text, p_state integer) returns table(nid_usuario integer)  
    language plpgsql  
as  
$$  
BEGIN  
    RETURN QUERY
        SELECT            
	        u.nid_usuario  
        FROM usuarios u  
        WHERE u.semail = p_email  
          AND u.nestado = p_state;  
  
end;  
$$;
```