```sql
create function get_preference_by_collaborator(nid_collaborator integer)  
returns setof preferenciascolaborador  
    language plpgsql  
as  
$$  
BEGIN  
    RETURN QUERY    SELECT        nid_preferencias,  
        ntheme,  
        scards_orden,  
        surl,  
        nid_colaborador,  
        ddatetime_creador,  
        ddatetime_update,  
        nusuario_creador,  
        nusuario_update,  
        nbackground_theme,  
        nmode_type_theme  
    FROM preferenciascolaborador  
    WHERE nid_colaborador = nid_collaborator;  
END;  
$$;
```