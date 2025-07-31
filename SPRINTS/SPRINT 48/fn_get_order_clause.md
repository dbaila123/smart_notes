```sql
create function fn_get_order_clause(p_campo text, p_order text, p_default text) returns text  
    language plpgsql  
as  
$$  
declare  
begin  
    if p_campo is not null and p_order is not null and trim(p_campo) <> '' and trim(p_order) <> '' then  
        if p_campo <> p_default then  
            return ' order by ' || p_campo || ' ' || p_order || ',' || p_default || ' asc';  
        else  
            return ' order by ' || p_campo || ' ' || p_order;  
        end if;  
    else  
        return ' order by ' || p_default || ' asc';  
    end if;  
end;  
$$;
```