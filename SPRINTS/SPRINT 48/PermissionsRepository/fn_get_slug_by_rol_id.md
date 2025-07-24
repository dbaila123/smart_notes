```sql
create function fn_get_slug_by_rol_id(p_nid_role integer, p_state integer) returns table(slug varchar(450))  
    language plpgsql  
as  
$$  
BEGIN  
    RETURN QUERY        
	    SELECT o.sslug  
        FROM permisos_opciones po  
        JOIN opciones o ON po.nid_opcion = o.nid_opcion  
        WHERE po.nid_rol = p_nid_role AND o.nestado = p_state;  
end;  
$$;
```