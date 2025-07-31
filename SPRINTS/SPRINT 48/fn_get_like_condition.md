```sql
create or replace function fn_get_like_condition(campo text, filter text) returns text  
    language plpgsql  
as  
$$  
BEGIN  
    RETURN campo || ' ILIKE ' || quote_literal('%' || filter || '%');  
END;  
$$;
```