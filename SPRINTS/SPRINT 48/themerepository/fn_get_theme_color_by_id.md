```sql
  
create function fn_get_theme_color_by_id(p_nid_color integer)  
returns setof tema_colores  
    language plpgsql  
as  
$$  
BEGIN  
    RETURN QUERY    
    SELECT        
		nid_color,  
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
    WHERE nid_color = p_nid_color;  
END;  
$$;
```