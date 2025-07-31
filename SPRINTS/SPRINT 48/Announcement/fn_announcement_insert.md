```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_announcement_insert(
	par_stitulo_anuncio character varying, 
	par_smensaje_anuncio character varying, 
	par_dfecha_inicio date, 
	par_dfecha_fin date, 
	par_nusuario_creador integer, 
	par_ddatetime_creador timestamp without time zone, 
	par_sefecto_animacion integer DEFAULT NULL::integer, 
	par_nusuario_update integer DEFAULT NULL::integer, 
	par_ddatetime_update timestamp without time zone DEFAULT NULL::timestamp without time zone
)
RETURNS integer
LANGUAGE plpgsql
AS $$
DECLARE
    v_id INT;
BEGIN
    INSERT INTO tenant0001_dbo.celebracion (
        stitulo_anuncio, 
        smensaje_anuncio, 
        sefecto_animacion, 
        dfecha_inicio, 
        dfecha_fin,
        nusuario_creador, 
        ddatetime_creador, 
        nusuario_update, 
        ddatetime_update
    )
    VALUES (
        par_stitulo_anuncio, 
        par_smensaje_anuncio, 
        par_sefecto_animacion, 
        par_dfecha_inicio, 
        par_dfecha_fin,
        par_nusuario_creador, 
        par_ddatetime_creador, 
        par_nusuario_update, 
        par_ddatetime_update
    )
    RETURNING nid_celebracion INTO v_id;

    RETURN v_id;
END;
$$;
```