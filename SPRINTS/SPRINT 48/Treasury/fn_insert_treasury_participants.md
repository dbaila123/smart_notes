```SQL
CREATE OR REPLACE FUNCTION fn_insert_treasury_participants(
    par_acolaboradores TEXT,
    par_nid_colaborador_principal TEXT,
    par_nid_solicitud_tesoreria INT,
    par_nid_proyecto INT DEFAULT NULL,
    par_nid_requerimiento INT DEFAULT NULL
)
RETURNS VOID
LANGUAGE plpgsql
AS $$
DECLARE
    var_ddatetime_creador TIMESTAMP := (NOW() AT TIME ZONE 'UTC') - INTERVAL '5 hours';
    var_all_collaborators INT[];
    var_principal_collaborators INT[];
    var_combined_collaborators INT[];
BEGIN
    var_all_collaborators := string_to_array(par_acolaboradores, ',')::INT[];
    var_principal_collaborators := string_to_array(par_nid_colaborador_principal, ',')::INT[];
    
    SELECT array_agg(DISTINCT id)
    INTO var_combined_collaborators
    FROM (
        SELECT unnest(var_all_collaborators || var_principal_collaborators) AS id
    ) subq
    WHERE id != 0;
    
    INSERT INTO solicitudes_tesoreria_participantes (
        nid_colaborador,
        nid_proyecto,
        nid_requerimiento,
        nid_solicitud_tesoreria,
        ddatetime_creador,
        bescolaboradorprincipal
    )
    SELECT 
        id,
        CASE WHEN par_nid_proyecto = 0 THEN NULL ELSE par_nid_proyecto END,
        CASE WHEN par_nid_requerimiento = 0 THEN NULL ELSE par_nid_requerimiento END,
        par_nid_solicitud_tesoreria,
        var_ddatetime_creador,
        CASE WHEN id = ANY(var_principal_collaborators) THEN 1 ELSE 0 END
    FROM 
        unnest(var_combined_collaborators) AS id;
END;
$$;
```