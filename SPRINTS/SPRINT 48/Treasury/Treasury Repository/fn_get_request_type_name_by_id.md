```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_request_type_name_by_id(par_request_type_id INT)
RETURNS VARCHAR
LANGUAGE plpgsql
AS $$
DECLARE v_name VARCHAR;
BEGIN
    SELECT sdescripcion
    INTO v_name
    FROM tenant0001_dbo.tesoreria_tipos_solicitud
    WHERE nid_tipo_solicitud = par_request_type_id;

    RETURN v_name;
END;
$$;
```