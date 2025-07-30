```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_order_clause(campo text, "order" text, "default" text)
 RETURNS text
 LANGUAGE plpgsql
AS $$
DECLARE
    result TEXT;
BEGIN
    IF campo IS NOT NULL AND "order" IS NOT NULL AND TRIM(campo) <> '' AND TRIM("order") <> '' THEN
        IF campo <> "default" THEN
            result := CONCAT('ORDER BY ', quote_ident(campo), ' ', "order", ', ', quote_ident("default"), ' ASC');
        ELSE
            result := CONCAT('ORDER BY ', quote_ident(campo), ' ', "order");
        END IF;
    ELSE
        result := CONCAT('ORDER BY ', quote_ident("default"), ' ASC');
    END IF;

    RETURN result;
END;
$$;
```