```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_requests_for_card(
    par_nid_colaborador INT,
    par_nanulado INT
)
RETURNS TABLE (
    nid_solicitud_tesoreria INT,
    snombre_tipo_solicitud TEXT,
    dfecha_efecto DATE,
    dfecha_creacion TIMESTAMP,
    nestado_solicitud INT,
    sestado_solicitud TEXT
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        sol_tes.nid_solicitud_tesoreria,
        config.sdescripcion,
        sol_tes.dfecha_efecto,
        sol_tes.ddatetime_creador,
        sol_tes.nestado_solicitud,
        config2.sdescripcion
    FROM tenant0001_dbo.solicitud_tesoreria sol_tes
    JOIN tenant0001_dbo.tesoreria_tipos_solicitud config 
        ON sol_tes.nid_tipo_solicitud = config.nid_tipo_solicitud
    JOIN tenant0001_dbo.configs config2 
        ON sol_tes.nestado_solicitud = config2.scodigo::INT
    WHERE sol_tes.nid_colaborador = par_nid_colaborador
      AND config2.stabla = 'ESTADO_SOLICITUD_TESORERIA'
      AND sol_tes.nestado_solicitud != par_nanulado
    ORDER BY sol_tes.dfecha_efecto DESC;
END;
$$;
```