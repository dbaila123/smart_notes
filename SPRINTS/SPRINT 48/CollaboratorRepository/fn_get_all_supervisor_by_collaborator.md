```sql
create or replace function fn_get_all_supervisor_by_collaborator(  
    p_collaborator_id integer,  
    p_supervisor_sec integer default null  
)  
    returns TABLE(nid_collaborator integer, nid_supervisor integer, norden integer)  
    language plpgsql  
as  
$$  
begin  
    RETURN QUERY    with recursive cte_supervisor as (  
        SELECT  
            cs.nid_colaborador,  
            cs.nid_supervisor,  
            1 as norden  
        FROM colaboradores_supervisor cs  
        WHERE cs.nid_colaborador = p_collaborator_id  
        and cs.bprincipal = 1  
  
        UNION ALL  
  
        select            nid_colaborador,  
            fn_get_id_supervisor_By_Collaborator(cs2.nid_supervisor, p_supervisor_sec),  
            cs2.norden + 1  
        FROM cte_supervisor cs2  
        WHERE fn_get_id_supervisor_By_Collaborator(cs2.nid_supervisor, p_supervisor_sec)  
            IS NOT NULL  
    )  
    SELECT  
        c.nid_colaborador,  
        c.nid_supervisor,  
        c.norden  
    FROM cte_supervisor c;  
end;  
$$;
```