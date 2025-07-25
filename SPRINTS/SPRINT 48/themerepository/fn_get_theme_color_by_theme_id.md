```SQL
create function fn_get_theme_color_by_theme_id(p_theme_id integer)  
returns setof tema_colores  
    language plpgsql  
as  
$$  
BEGIN  
    RETURN QUERY    SELECT       nid_color,  
        nid_tema,  
        dfecha_inicio,  
        dfecha_fin,  
        scolores,  
        nestado,  
        nusuario_creador,  
        ddatetime_creador,  
        nusuario_update,  
        ddatetime_update,  
        nusuario_delete,  
        ddatetime_delete  
    FROM tema_colores  
    WHERE nid_color = p_theme_id;  
END;  
$$;