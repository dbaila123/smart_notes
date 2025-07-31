```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_pagination_clause(numpage integer, records integer)
 RETURNS text
 LANGUAGE plpgsql
AS $$
DECLARE
    result TEXT;
BEGIN
    IF numpage IS NOT NULL AND records IS NOT NULL THEN
        result := CONCAT(
            'LIMIT ', records, ' OFFSET ', (numpage - 1) * records
        );
    ELSE
        result := '';
    END IF;

    RETURN result;
END;
$$;
```