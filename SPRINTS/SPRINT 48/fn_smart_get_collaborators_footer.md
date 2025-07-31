```SQL
create or replace function fn_smart_get_collaborators_footer(p_nid integer, p_collaborator_name text, p_slug_to_get_list text, p_start_date text, p_end_date text, p_filter_one text, p_filter_two text, p_filter_three text, p_filter_four text, p_filter_five text, p_filter_six text)  
    returns TABLE(colaboradores_activos integer, colaboradores_cesados integer)  
    language plpgsql  
as  
$$  
DECLARE  
    var_view_clause TEXT;  
    var_where_clause TEXT := '';  
    var_conditions TEXT[] := ARRAY[]::TEXT[];  
    var_condition TEXT;  
    var_script TEXT;  
    var_collaborator_active_script TEXT;  
    var_collaborator_ceased_script TEXT;  
    var_state_collaborator_active TEXT := '1';  
    var_state_collaborator_ceased TEXT := '2';  
    var_collaborator_list TEXT;  
    var_collaborator_list_team TEXT;  
    var_collaborators_active INTEGER;  
    var_collaboators_ceased INTEGER;  
    i INTEGER;  
BEGIN  
    -- Tipo de listado  
    IF p_slug_to_get_list = 'COLABORADOR-LISTA-INDIVIDUAL' THEN  
        var_view_clause := 'select * from (select * from v_collaborator_list where nid = ' || p_nid || ') a';  
    ELSIF p_slug_to_get_list = 'TEAM-HAVE-SUBORDINATE' THEN  
        SELECT fn_get_collaborators_by_supervisor(p_nid::TEXT, 0, 0) INTO var_collaborator_list;  
        var_view_clause := 'select * from (select * from v_collaborator_list where nid in (' || var_collaborator_list || ')) a';  
    ELSIF p_slug_to_get_list = 'COLABORADOR-LISTADO' THEN  
        var_view_clause := 'select * from v_collaborator_list';  
    END IF;  
  
    -- Filtros  
    IF p_collaborator_name IS NOT NULL THEN  
        var_condition := fn_get_like_condition('snombre_colaborador', p_collaborator_name);  
        var_conditions := array_append(var_conditions, var_condition);  
    END IF;  
  
    -- Filtros por fechas  
    IF p_start_date IS NOT NULL AND p_end_date IS NOT NULL THEN  
        var_condition := fn_get_dates_condition('dfec_ingreso', p_start_date, p_end_date);  
        var_conditions := array_append(var_conditions, var_condition);  
    END IF;  
  
    IF p_filter_one IS NOT NULL THEN  
        var_condition := 'ntipo_modalidad in (' || p_filter_one || ')';  
        RAISE NOTICE 'tipo modalidad';  
        var_conditions := array_append(var_conditions, var_condition);  
    END IF;  
  
    IF p_filter_two IS NOT NULL THEN  
        var_condition := 'nestado in (' || p_filter_two || ')';  
        RAISE NOTICE 'tipo estado';  
        var_conditions := array_append(var_conditions, var_condition);  
    END IF;  
  
    IF p_filter_three IS NOT NULL THEN  
        SELECT fn_get_collaborators_by_supervisor(p_filter_three, 0, 0) INTO var_collaborator_list_team;  
        var_condition := 'nid IN (' || var_collaborator_list_team || ')';  
        RAISE NOTICE 'equipo';  
        var_conditions := array_append(var_conditions, var_condition);  
    END IF;  
  
    if p_filter_four is not null then  
        var_condition := 'nid_sede in (' || p_filter_four || ')';  
        var_conditions := array_append(var_conditions, var_condition);  
    end if;  
  
    if p_filter_five is not null then  
        var_condition := 'nid_cargo in (' || p_filter_five || ')';  
        var_conditions := array_append(var_conditions, var_condition);  
    end if;  
  
    if p_filter_six is not null then  
        var_condition := 'nid_empresa in (' || p_filter_six || ')';  
        var_conditions := array_append(var_conditions, var_condition);  
    end if;  
  
    -- Construir WHERE clause  
    IF array_length(var_conditions, 1) > 0 THEN  
        var_where_clause := 'WHERE ' || array_to_string(var_conditions, ' AND ');  
    END IF;  
  
    -- Scripts  
    var_script := var_view_clause || ' ' || var_where_clause;  
    var_collaborator_active_script := 'select count(*) from (' || var_script || ') a where a.nestado = ' || var_state_collaborator_active;  
    var_collaborator_ceased_script := 'select count(*) from (' || var_script || ') a where a.nestado = ' || var_state_collaborator_ceased;  
  
    RAISE NOTICE '%', var_script;  
    RAISE NOTICE '%', var_collaborator_active_script;  
    RAISE NOTICE '%', var_collaborator_ceased_script;  
  
    -- Ejecutar consultas dinámicas  
    EXECUTE var_collaborator_active_script INTO var_collaborators_active;  
    EXECUTE var_collaborator_ceased_script INTO var_collaboators_ceased;  
  
    -- Retornar resultados  
    RETURN QUERY SELECT var_collaborators_active, var_collaboators_ceased;  
END;  
$$;