```SQL
CREATE FUNCTION fn_get_themes_by_date(p_today timestamp)  
RETURNS SETOF Temas  
LANGUAGE plpgsql  
AS  
$$  
BEGIN  
    RETURN QUERY    SELECT *  
    FROM Temas  
    WHERE dFecha_Inicio <= p_today  
      AND dFecha_Fin >= p_today;  
END;  
$$;
```