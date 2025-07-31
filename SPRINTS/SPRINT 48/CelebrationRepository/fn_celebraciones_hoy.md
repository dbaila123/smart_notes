```SQL
CREATE OR REPLACE FUNCTION fn_celebraciones_hoy()
RETURNS TABLE (
    nid_usuario    INTEGER,
    snombre_corto  TEXT,
    surl_foto      TEXT,
    nyears         INTEGER,
    sestado        TEXT
)
LANGUAGE plpgsql
AS $$
DECLARE
    var_month INTEGER := EXTRACT(MONTH FROM clock_timestamp() AT TIME ZONE 'UTC-5');
    var_day   INTEGER := EXTRACT(DAY   FROM clock_timestamp() AT TIME ZONE 'UTC-5');
BEGIN
    RETURN QUERY
    SELECT
        v.nid_usuario,
        v.snombre_corto,
        v.surl_foto,
        v.nyears,
        v.sestado
    FROM v_celebraciones v
    WHERE
        (
            date_part('month', v.dfecha_nacimiento) = var_month
            AND date_part('day', v.dfecha_nacimiento) = var_day
            AND v.ncodigo_estado = 1
        )
        OR
        (
            date_part('month', v.dfecha_ingreso) = var_month
            AND date_part('day', v.dfecha_ingreso) = var_day
            AND v.ncodigo_estado = 2
        );
END;
$$;