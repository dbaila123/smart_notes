```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_category_by_request_type(par_request_type_id INT)
RETURNS INT
LANGUAGE plpgsql
AS $$
DECLARE v_categoria INT;
BEGIN
    SELECT tc.nid_categoria
    INTO v_categoria
    FROM tenant0001_dbo.tesoreria_tipos_solicitud tt
    JOIN tenant0001_dbo.tesoreria_categorias tc 
        ON tt.nid_categoria = tc.nid_categoria
    WHERE tt.nid_tipo_solicitud = par_request_type_id;

    RETURN v_categoria;
END;
$$;
```