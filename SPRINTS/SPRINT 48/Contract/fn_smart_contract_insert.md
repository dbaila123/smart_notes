```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_smart_contract_insert(
	par_nid_colaborador integer, 
	par_nid_cargo integer, 
	par_dfecha_ini timestamp without time zone, 
	par_dfecha_fin timestamp without time zone, 
	par_dsalario numeric, 
	par_sobservacion text, 
	par_nestado_contrato integer, 
	par_nestado integer, 
	par_nusuario_creador integer, 
	par_ddatetime_creador timestamp without time zone, 
	par_dfecha_cese timestamp without time zone DEFAULT NULL::timestamp without time zone, 
	par_smotivo_cese text DEFAULT NULL::text, 
	par_nusuario_update integer DEFAULT NULL::integer, 
	par_ddatetime_update timestamp without time zone DEFAULT NULL::timestamp without time zone, 
	par_nusuario_delete integer DEFAULT NULL::integer, 
	par_ddatetime_delete timestamp without time zone DEFAULT NULL::timestamp without time zone
)
RETURNS integer
LANGUAGE plpgsql
AS $$
DECLARE
    v_nid_contrato INTEGER := 0;
BEGIN
    INSERT INTO contratos (
        nid_colaborador,
        nid_cargo,
        dfecha_ini,
        dfecha_fin,
        dsalario,
        sobservacion,
        dfecha_cese,
        smotivo_cese,
        nestado_contrato,
        nestado,
        nusuario_creador,
        ddatetime_creador,
        nusuario_update,
        ddatetime_update,
        nusuario_delete,
        ddatetime_delete
    )
    VALUES (
        par_nid_colaborador,
        par_nid_cargo,
        par_dfecha_ini,
        par_dfecha_fin,
        par_dsalario,
        par_sobservacion,
        par_dfecha_cese,
        par_smotivo_cese,
        par_nestado_contrato,
        par_nestado,
        par_nusuario_creador,
        par_ddatetime_creador,
        par_nusuario_update,
        par_ddatetime_update,
        par_nusuario_delete,
        par_ddatetime_delete
    )
    RETURNING nid_contrato INTO v_nid_contrato;

    RETURN v_nid_contrato;
END;
$$;
```