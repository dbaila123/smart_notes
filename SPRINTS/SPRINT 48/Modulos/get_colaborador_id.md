```POSTGRESQL
CREATE OR REPLACE FUNCTION get_colaborador_id(nId_User INTEGER)  
RETURNS INTEGER AS $$  
DECLARE  
    result INTEGER;  
BEGIN  
    SELECT c.nId_Colaborador  
    INTO result  
    FROM Usuarios u  
    JOIN Personas p ON u.nId_Persona = p.nId_Persona  
    JOIN Colaboradores c ON p.nId_Persona = c.nId_Persona  
    WHERE u.nId_Usuario = nId_User;  
  
    RETURN result;  
END;  
$$ LANGUAGE plpgsql;
```