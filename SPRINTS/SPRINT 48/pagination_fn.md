```sql
create or replace function fn_get_pagination_clause(p_num_page integer, p_records integer)  
    returns text  
    language plpgsql  
as  
$$  
begin  
    if p_num_page is not null and p_records is not null then  
        return 'LIMIT ' || p_records || ' OFFSET ' || (p_num_page - 1) * p_records;  
    end if;  
  
    return '';  
end;  
	$$;
```