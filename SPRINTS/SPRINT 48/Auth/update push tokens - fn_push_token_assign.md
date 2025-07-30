```sql
create oe replace function fn_push_token_assign(p_sauth text, p_nid_usuario integer) returns integer  
    language plpgsql  
as  
$$  
DECLARE  
    updated_rows integer;  
BEGIN  
    IF EXISTS (SELECT 1 FROM tenant0001_dbo.push_tokens WHERE nid_usuario = p_nId_Usuario) THEN  
        UPDATE tenant0001_dbo.push_tokens  
        SET  
            nid_usuario = NULL,  
            nestado = 0  
        WHERE nid_usuario = p_nId_Usuario;  
    END IF;  
  
    UPDATE tenant0001_dbo.push_tokens  
    SET  
        nid_usuario = p_nId_Usuario,  
        nestado = 1  
    WHERE LOWER(sauth) = LOWER(p_sauth);  
  
    GET DIAGNOSTICS updated_rows = ROW_COUNT;  
    RETURN updated_rows;  
END;  
$$;
```