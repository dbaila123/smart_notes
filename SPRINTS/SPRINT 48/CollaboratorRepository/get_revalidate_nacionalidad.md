```sql
CREATE OR REPLACE FUNCTION fn_get_revalidate_nacionalidad(p_gentilicio TEXT)
RETURNS TEXT AS $$
DECLARE
    resultado TEXT;
BEGIN
    SELECT sgentilicio
    INTO resultado
    FROM paises
    WHERE LOWER(sgentilicio) = LOWER(p_gentilicio)
      AND nestado = 1
    LIMIT 1;

    RETURN resultado;
END;
$$ LANGUAGE plpgsql;