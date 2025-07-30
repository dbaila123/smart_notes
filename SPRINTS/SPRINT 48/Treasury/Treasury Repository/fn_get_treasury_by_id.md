```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_treasury_by_id(par_nid_solicitud INT)
RETURNS TABLE (
    nid_solicitud_tesoreria INT,
    nid_colaborador INT,
    nid_tipo_solicitud INT,
    dfecha_efecto DATE,
    ddatetime_creador TIMESTAMP,
    nestado_solicitud INT,
    nestado INT
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        s.nid_solicitud_tesoreria,
        s.nid_colaborador,
        s.nid_tipo_solicitud,
        s.dfecha_efecto,
        s.ddatetime_creador,
        s.nestado_solicitud,
        s.nestado
    FROM tenant0001_dbo.solicitud_tesoreria s
    WHERE s.nid_solicitud_tesoreria = par_nid_solicitud;
END;
$$;
```