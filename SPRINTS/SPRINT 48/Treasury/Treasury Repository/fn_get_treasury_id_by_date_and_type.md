```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_treasury_id_by_date_and_type(
    par_dfecha_efecto DATE,
    par_nid_colaborador INT,
    par_ntipo_solicitud INT,
    par_nrejected INT,
    par_ncancelled INT
)
RETURNS INT
LANGUAGE plpgsql
AS $$
DECLARE v_id INT;
BEGIN
    SELECT nid_solicitud_tesoreria
    INTO v_id
    FROM tenant0001_dbo.solicitud_tesoreria
    WHERE dfecha_efecto = par_dfecha_efecto
      AND nid_colaborador = par_nid_colaborador
      AND nid_tipo_solicitud = par_ntipo_solicitud
      AND nestado = 1
      AND nestado_solicitud NOT IN (par_nrejected, par_ncancelled)
    ORDER BY nid_solicitud_tesoreria DESC
    LIMIT 1;

    RETURN v_id;
END;
$$;
```