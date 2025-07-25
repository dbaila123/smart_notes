```SQL
create function fn_get_background_by_theme_id(p_theme_id integer) returns SETOF tema_fondos  
    language plpgsql  
as  
$$  
BEGIN  
    RETURN QUERY    SELECT        nid_fondo,  
        nid_tema,  
        dfecha_inicio,  
        dfecha_fin,  
        sfondo,  
        sicono,  
        nestado,  
        nusuario_creador,  
        ddatetime_creador,  
        nusuario_update,  
        ddatetime_update,  
        nusuario_delete,  
        ddatetime_delete  
    FROM tema_fondos  
    WHERE nid_tema = p_theme_id;  
END;  
$$;