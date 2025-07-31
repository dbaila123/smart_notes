```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_contract_update(
	in_nid_contrato integer, 
	in_nid_colaborador integer, 
	in_nid_cargo integer DEFAULT NULL::integer, 
	in_dfecha_ini timestamp without time zone DEFAULT NULL::timestamp without time zone, 
	in_dfecha_fin timestamp without time zone DEFAULT NULL::timestamp without time zone, 
	in_dsalario numeric DEFAULT NULL::numeric, 
	in_sobservacion text DEFAULT NULL::text, 
	in_dfecha_cese timestamp without time zone DEFAULT NULL::timestamp without time zone, 
	in_smotivo_cese text DEFAULT NULL::text, 
	in_nestado_contrato integer DEFAULT NULL::integer, 
	in_nestado integer DEFAULT NULL::integer, 
	in_nusuario_creador integer DEFAULT NULL::integer, 
	in_ddatetime_creador timestamp without time zone DEFAULT NULL::timestamp without time zone, 
	in_nusuario_update integer DEFAULT NULL::integer, 
	in_ddatetime_update timestamp without time zone DEFAULT NULL::timestamp without time zone, 
	in_nusuario_delete integer DEFAULT NULL::integer, 
	in_ddatetime_delete timestamp without time zone DEFAULT NULL::timestamp without time zone
)
RETURNS void
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE contratos
    SET
        nid_cargo = COALESCE(in_nid_cargo, nid_cargo),
        dfecha_ini = COALESCE(in_dfecha_ini, dfecha_ini),
        dfecha_fin = COALESCE(in_dfecha_fin, dfecha_fin),
        dsalario = COALESCE(in_dsalario, dsalario),
        sobservacion = COALESCE(in_sobservacion, sobservacion),
        dfecha_cese = COALESCE(in_dfecha_cese, dfecha_cese),
        smotivo_cese = COALESCE(in_smotivo_cese, smotivo_cese),
        nestado_contrato = COALESCE(in_nestado_contrato, nestado_contrato),
        nestado = COALESCE(in_nestado, nestado),
        nusuario_creador = COALESCE(in_nusuario_creador, nusuario_creador),
        ddatetime_creador = COALESCE(in_ddatetime_creador, ddatetime_creador),
        nusuario_update = COALESCE(in_nusuario_update, nusuario_update),
        ddatetime_update = COALESCE(in_ddatetime_update, ddatetime_update),
        nusuario_delete = COALESCE(in_nusuario_delete, nusuario_delete),
        ddatetime_delete = COALESCE(in_ddatetime_delete, ddatetime_delete)
    WHERE nid_contrato = in_nid_contrato AND nid_colaborador = in_nid_colaborador;
END;
$$;
```