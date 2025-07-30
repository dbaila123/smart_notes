```sql
create function fn_get_collaborators_by_supervisor(  
    p_id_supervisors text,  
    b_only_direct_collaborators numeric(1),  
    b_only_principal numeric(1))  
returns text  
    language plpgsql  
AS $function$  
DECLARE  
    var_result text;  
    var_asupervisors integer[];  
    var_nid_supervisor integer;  
    var_collaborators integer[];  
begin  
    var_asupervisors := string_to_array(p_id_supervisors, ',');  
  
    FOREACH var_nid_supervisor IN ARRAY var_asupervisors  
    LOOP  
        var_collaborators := var_collaborators || var_nid_supervisor;  
  
        if b_only_direct_collaborators = 1 then  
            var_collaborators :=  
                var_collaborators ||  
                ARRAY(  
                  SELECT DISTINCT  
                      nid_colaborador  
                  FROM colaboradores_supervisor  
                  WHERE nid_supervisor = var_nid_supervisor  
                  AND nestado = 1  
                  AND bprincipal = (  
                      CASE  
                          WHEN b_only_principal = 1 THEN b_only_principal  
                      ELSE bprincipal  
                      END  
                      )  
                );  
        else  
            --text  
            var_collaborators := var_collaborators || (  
                WITH RECURSIVE subordinates_recursive AS (  
                    SELECT DISTINCT  
                        cs.nId_Colaborador  
                    FROM colaboradores_supervisor cs  
                    WHERE cs.nId_Supervisor = ANY(var_collaborators)  
                      AND cs.nEstado = 1  
                      AND cs.bPrincipal = (  
                          CASE  
                              WHEN b_only_principal = 1 THEN b_only_principal  
                          ELSE cs.bPrincipal  
                          END  
                          )  
                    UNION  
  
                    SELECT                        cs2.nId_Colaborador  
                    FROM colaboradores_supervisor cs2  
                    JOIN subordinates_recursive cr ON cs2.nId_Supervisor = cr.nId_Colaborador  
                    WHERE cs2.nEstado = 1  
                        AND cs2.bprincipal = (  
                            CASE  
                                WHEN b_only_principal = 1 THEN b_only_principal  
                            ELSE cs2.bprincipal  
                            END  
                            )  
                )  
                SELECT array_agg(DISTINCT nid_colaborador)  
                FROM subordinates_recursive  
            );  
            EXIT;  
        end if;  
    end loop;  
  
    var_result := array_to_string(var_collaborators, ',');  
  
    return var_result;  
end;  
$function$;
```