```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_like_condition(campo text, filter text)
 RETURNS text
 LANGUAGE plpgsql
AS $$
BEGIN
    RETURN CONCAT(campo, ' ILIKE ''%', filter, '%''');
END;
$$;
```