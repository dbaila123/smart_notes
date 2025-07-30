```SQL
create or replace function fn_smart_get_session_by_user(p_nid_user integer) returns SETOF sesiones  
    language plpgsql  
as  
$$  
BEGIN  
    RETURN QUERY    SELECT        s.nid_usuario,  
        s.stoken_usuario,  
        s.nip,  
        s.nso,  
        s.ndispositivo,  
        s.nnavegador,  
        s.nestado,  
        s.nusuario_creador,  
        s.ddatetime_creador,  
        s.nusuario_update,  
        s.ddatetime_update,  
        s.nusuario_delete,  
        s.ddatetime_delete  
    FROM sesiones s  
    WHERE s.nid_usuario = p_nid_user;  
END  
$$;
```