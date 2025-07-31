```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_dates_condition(campo text, dateini text, datefin text)
 RETURNS text
 LANGUAGE plpgsql
AS $$
BEGIN
    dateini := dateini || ' 00:00:00.0000000';
    datefin := datefin || ' 23:59:59.9999999';
    
    RETURN CONCAT(campo, ' BETWEEN ''', dateini::TIMESTAMP, ''' AND ''', datefin::TIMESTAMP, '''');
END;
$$;
```