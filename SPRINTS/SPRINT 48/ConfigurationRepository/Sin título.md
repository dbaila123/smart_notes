```sql
create function function_IdUserORIdColabordor(p_valor text, p_id integer)  
returns integer  
    LANGUAGE plpgsql  
AS $function$  
DECLARE  
    var_valor integer;  
begin  
    if p_valor = 'usuario' then  
        var_valor := (SELECT c.nid_colaborador  
                    FROM usuarios u  
                    JOIN personas p on p.nid_persona = u.nid_persona  
                    JOIN colaboradores c on c.nid_persona = p.nid_persona  
                    WHERE u.nid_usuario = p_id  
                    );  
    end if;  
    if p_valor = 'colaborador' then  
        var_valor := (SELECT u.nid_usuario  
                      FROM usuarios u  
                               JOIN personas p on p.nid_persona = u.nid_persona  
                               JOIN colaboradores c on c.nid_persona = p.nid_persona  
                      WHERE c.nid_colaborador = p_id );  
    end if;  
  
    return var_valor;  
end;  
$function$;
```