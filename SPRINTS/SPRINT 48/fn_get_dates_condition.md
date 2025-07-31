```sql
create or replace function fn_get_dates_condition(  
    p_campo text,  
    p_date_ini text,  
    p_date_fin text  
    )  
    returns text  
    language plpgsql  
as  
$$  
DECLARE  
    var_date_ini timestamp;  
    var_date_fin timestamp;  
BEGIN  
    var_date_ini := (p_date_ini || ' 00:00:00.000000')::timestamp;  
    var_date_fin := (p_date_fin || ' 23:59:59.999999')::timestamp;  
  
    return p_campo || ' BETWEEN ' ||  
                     quote_literal(var_date_ini) || ' AND ' ||  
                     quote_literal(var_date_fin);  
END;  
$$;
```