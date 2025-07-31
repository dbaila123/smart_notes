```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_validar_plantilla(
    par_sid_drivefile TEXT,
    par_stipo_contratacion TEXT,
    par_nempresa_usuaria INT,
    par_create_mode BOOLEAN
)
RETURNS BOOLEAN
LANGUAGE plpgsql
AS $$
DECLARE
    exists_count INT;
BEGIN
    IF par_create_mode THEN
        SELECT COUNT(1)
        INTO exists_count
        FROM tenant0001_dbo.contract_templates
        WHERE sid_drivefile = par_sid_drivefile
           OR (stipo_contratacion = par_stipo_contratacion 
               AND nempresa_usuaria = par_nempresa_usuaria);
    ELSE
        SELECT COUNT(1)
        INTO exists_count
        FROM tenant0001_dbo.contract_templates
        WHERE stipo_contratacion = par_stipo_contratacion 
          AND nempresa_usuaria = par_nempresa_usuaria;
    END IF;

    RETURN exists_count > 0;
END;
$$;
```