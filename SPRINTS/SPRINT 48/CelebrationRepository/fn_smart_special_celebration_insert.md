```SQL
CREATE OR REPLACE FUNCTION fn_smart_special_celebration_insert(
    p_stitulo_anuncio     TEXT DEFAULT '',
    p_smensaje_anuncio    TEXT DEFAULT '',
    p_sefecto_animacion   INTEGER DEFAULT NULL,
    p_dfecha_inicio       TIMESTAMP DEFAULT NULL,
    p_dfecha_fin          TIMESTAMP DEFAULT NULL,
    p_nusuario_creador    INTEGER DEFAULT NULL,
    p_ddatetime_creador   TIMESTAMP DEFAULT NULL,
    p_nusuario_update     INTEGER DEFAULT NULL,
    p_ddatetime_update    TIMESTAMP DEFAULT NULL
)
RETURNS INTEGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_nid_celebracion INTEGER;
BEGIN
    INSERT INTO celebracion (
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
        COALESCE(p_stitulo_anuncio, ''),
        COALESCE(p_smensaje_anuncio, ''),+
        p_sefecto_animacion,
        p_dfecha_inicio,
        p_dfecha_fin,
        p_nusuario_creador,
        p_ddatetime_creador,
        p_nusuario_update,
        p_ddatetime_update
    )
    RETURNING nid_celebracion INTO v_nid_celebracion;

    RETURN v_nid_celebracion;
END;
$$;