create function collaborator_suggests_by_slug(p_slug_listado text, p_collaborator_id integer)  
    returns table (scodigo text, sdescription text, stipo_configuration text)  
    language plpgsql  
AS $function$  
DECLARE  
    var_collaborator_by_supervisor integer[];  
    var_collaborator text;  
    var_nid_projects integer[];  
begin  
    if p_slug_listado = 'TODOS' OR p_slug_listado = '' then  
        RETURN QUERY        SELECT            c.nid_colaborador::text,  
            p.spersona_nombre,  
            'COLABORADORES'  
        FROM colaboradores c  
        JOIN personas p on c.nid_persona = p.nid_persona  
        JOIN horarios_colaboradores hc ON c.nid_colaborador = hc.nid_colaborador  
        WHERE hc.nestado = 1;  
    else  
        var_collaborator := fn_get_collaborators_by_supervisor(p_collaborator_id::text, 0, 0);  
  
        var_collaborator_by_supervisor := ARRAY(  
                                            SELECT  
                                                elem::integer  
                                            FROM unnest(string_to_array(var_collaborator, ',')) as elem  
                                          );  
  
        var_nid_projects := ARRAY(  
                                SELECT DISTINCT  
                                    pl.nid_proyecto  
                                FROM proyecto_lider pl  
                                WHERE pl.nid_lider = p_collaborator_id  
                                    AND pl.nestado = 1  
                                UNION  
                                SELECT                                    p.nid_proyecto  
                                FROM proyectos p  
                                WHERE p.nid_director = p_collaborator_id  
                            );  
  
        RETURN QUERY  
        SELECT DISTINCT            c.nid_colaborador::text,  
            p.spersona_nombre,  
            'COLABORADORES'  
        FROM colaboradores c  
        JOIN personas p on c.nid_persona = p.nid_persona  
        JOIN tareas t on c.nid_colaborador = t.nid_colaborador  
        WHERE t.nid_proyecto = ANY(var_nid_projects)  
  
        UNION  
  
        SELECT DISTINCT            c1.nid_colaborador::text,  
            p1.spersona_nombre,  
            'COLABORADORES'  
        FROM colaboradores c1  
        JOIN personas p1 on c1.nid_persona = p1.nid_persona  
        WHERE c1.nid_colaborador = ANY(var_collaborator_by_supervisor);  
  
    end if;  
end;  
$function$;