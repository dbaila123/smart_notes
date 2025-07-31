```sql
create function fn_smart_get_collaborators(  
    p_nid integer,  
    p_collaborator_name text,  
    p_slug_to_get_list text,  
    p_start_date text,  
    p_end_date text,  
    p_filter_one text,  
    p_filter_two text,  
    p_filter_three text,  
    p_filter_four text,  
    p_filter_five text,  
    p_filter_six text,  
    p_pnum integer,  
    p_size integer,  
    p_column_to_order text,  
    p_order text  
    )  
    returns setof collaborator_list_response  
    language plpgsql  
as  
$$  
declare  
    var_script text;  
    var_view_clause text;  
    var_where_clause text;  
    var_order_clause text;  
    var_pagination_clause text;  
    var_count text;  
    var_collaborator_list text;  
    var_conditions_array text[];  
    var_condition text;  
begin  
    if p_slug_to_get_list = 'COLABORADOR-LISTA-INDIVIDUAL' then  
        var_view_clause := 'select * from (select * from v_collaborator_list where nid ='+p_nid::text+')a';  
    end if;  
  
    if p_slug_to_get_list = 'TEAM-HAVE-SUBORDINATE' then  
        var_collaborator_list :=  
            fn_get_collaborators_by_supervisor(p_nid::text, 0, 0);  
  
        var_view_clause := 'select * from (select * from v_collaborator_list where nid IN ( '+ var_collaborator_list +' )) a';  
    end if;  
  
    if p_slug_to_get_list = 'COLABORADOR-LISTADO' then  
        var_view_clause := 'select * from v_collaborator_list';  
    end if;  
  
    if p_collaborator_name is not null then  
        var_condition := fn_get_like_condition('snombre_colaborador', p_collaborator_name);  
        var_conditions_array := array_append(var_conditions_array, var_condition);  
    end if;  
  
    if p_start_date is not null and p_end_date is not null then  
        var_condition := fn_get_dates_condition('dfec_ingreso', p_start_date, p_end_date);  
        var_conditions_array := array_append(var_conditions_array, var_condition);  
    end if;  
  
    if p_filter_one is not null then  
        var_condition := 'ntipo_modalidad in (' || p_filter_one || ')';  
        var_conditions_array := array_append(var_conditions_array, var_condition);  
    end if;  
  
    if p_filter_two is not null then  
        var_condition := 'nestado in (' || p_filter_two || ')';  
        var_conditions_array := array_append(var_conditions_array, var_condition);  
    end if;  
  
    if p_filter_three is not null then  
        var_condition := fn_get_collaborators_by_supervisor(p_filter_three, 0, 0);  
        var_conditions_array := array_append(var_conditions_array, 'nid in (' || var_condition || ')');  
    end if;  
  
    if p_filter_four is not null then  
        var_condition := 'nid_sede in (' || p_filter_four || ')';  
        var_conditions_array := array_append(var_conditions_array, var_condition);  
    end if;  
  
    if p_filter_five is not null then  
        var_condition := 'nid_cargo in (' || p_filter_five || ')';  
        var_conditions_array := array_append(var_conditions_array, var_condition);  
    end if;  
  
    if p_filter_six is not null then  
        var_condition := 'nid_empresa in (' || p_filter_six || ')';  
        var_conditions_array := array_append(var_conditions_array, var_condition);  
    end if;  
  
    if array_length(var_conditions_array, 1) > 0 then  
        var_where_clause := 'where ' || array_to_string(var_conditions_array, ' and ');  
    end if;  
  
    var_order_clause := fn_get_order_clause(p_column_to_order, p_order, 'dfec_ingreso');  
  
end;  
$$;
```