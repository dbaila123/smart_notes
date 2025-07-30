```sql
create or replace function fn_get_supervisors_by_collaborators(  
    p_collaborators text default null,  
    p_supervisor_sec integer default null  
)  
    returns TABLE(nid_collaborator integer, nid_supervisor integer, norden integer)  
    language plpgsql  
as  
$$  
declare  
    var_collaborators integer[];  
    var_current_collaborator integer;  
begin  
    if p_collaborators is not null then  
        var_collaborators := string_to_array(p_collaborators, ',');  
    else  
        var_collaborators := ARRAY(  
                             SELECT v.nid  
                             FROM v_collaborator_list v  
                             );  
    end if;  
  
    foreach var_current_collaborator IN ARRAY var_collaborators LOOP  
        RETURN QUERY        SELECT f.nid_collaborator,  
               f.nid_supervisor,  
               f.norden  
        FROM fn_get_all_supervisor_by_collaborator(var_current_collaborator, p_supervisor_sec) f;  
    end loop;  
end;  
$$;
```