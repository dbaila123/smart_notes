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
            (p_session->>'nId_Usuario')::integer,  
            p_session->>'sToken_Usuario',  
            p_session->>'nIP',  
            p_session->>'nSO',  
            p_session->>'nDispositivo',  
            p_session->>'nNavegador',  
            (p_session->>'nEstado')::integer,  
            (p_session->>'nUsuario_Creador')::integer,  
            (p_session->>'dDatetime_Creador')::timestamp  
           );  
  
    GET DIAGNOSTICS inserted_rows = ROW_COUNT;  
    RETURN inserted_rows;  
END;  
$$;