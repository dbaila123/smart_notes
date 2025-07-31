```SQL
CREATE OR REPLACE FUNCTION fn_smart_special_celebration_update(
    p_nid_celebracion     INTEGER,
    p_stitulo_anuncio     TEXT DEFAULT NULL,
    p_smensaje_anuncio    TEXT DEFAULT NULL,
    p_dfecha_inicio       TIMESTAMP DEFAULT NULL,
    p_dfecha_fin          TIMESTAMP DEFAULT NULL,
    p_ddatetime_creador   TIMESTAMP DEFAULT NULL,
    p_ddatetime_update    TIMESTAMP DEFAULT NULL,
    p_nusuario_creador    INTEGER DEFAULT NULL,
    p_nusuario_update     INTEGER DEFAULT NULL,
    p_sefecto_animacion   INTEGER DEFAULT NULL
)
RETURNS TABLE(success INTEGER)
LANGUAGE plpgsql
AS $$
DECLARE
    affected_rows INTEGER;
BEGIN
    UPDATE celebracion
    SET
        stitulo_anuncio     = COALESCE(p_stitulo_anuncio, stitulo_anuncio),
        smensaje_anuncio    = COALESCE(p_smensaje_anuncio, smensaje_anuncio),
        dfecha_inicio       = COALESCE(p_dfecha_inicio, dfecha_inicio),
        dfecha_fin          = COALESCE(p_dfecha_fin, dfecha_fin),
        ddatetime_creador   = COALESCE(p_ddatetime_creador, ddatetime_creador),
        ddatetime_update    = COALESCE(p_ddatetime_update, ddatetime_update),
        nusuario_creador    = COALESCE(p_nusuario_creador, nusuario_creador),
        nusuario_update     = COALESCE(p_nusuario_update, nusuario_update),
        sefecto_animacion   = COALESCE(p_sefecto_animacion, sefecto_animacion)
    WHERE nid_celebracion = p_nid_celebracion;

    GET DIAGNOSTICS affected_rows = ROW_COUNT;
    RETURN QUERY SELECT CASE WHEN affected_rows > 0 THEN 1 ELSE 0 END;
END;
$$;