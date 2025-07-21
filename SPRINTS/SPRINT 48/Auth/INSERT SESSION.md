```SQL
create function sp_session_insert(p_session json) returns integer  
    language plpgsql  
as  
$$  
DECLARE  
    inserted_rows integer;  
BEGIN  
    INSERT INTO sesiones(nId_Usuario, sToken_Usuario, nIP, nSO, nDispositivo, nNavegador, nEstado, nUsuario_Creador, dDatetime_Creador)  
    VALUES (  
            (p_session->>'nid_usuario')::integer,  
            p_session->>'stoken_usuario',  
            p_session->>'nip',  
            p_session->>'nso',  
            p_session->>'ndispositivo',  
            p_session->>'nnavegador',  
            (p_session->>'nestado')::integer,  
            (p_session->>'nusuario_creador')::integer,  
            (p_session->>'ddatetime_creador')::timestamp  
           );  
  
    GET DIAGNOSTICS inserted_rows = ROW_COUNT;  
    RETURN inserted_rows;  
END;  
$$;
```