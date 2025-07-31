```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_all_announcements(
	par_current_date date
)
RETURNS TABLE(nid_celebracion integer, stitulo_anuncio character varying, smensaje_anuncio character varying, dfecha_inicio date, dfecha_fin date, ddatetime_creador timestamp without time zone, ddatetime_update timestamp without time zone, nusuario_creador integer, nusuario_update integer, sefecto_animacion integer, nestado integer)
 LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT 
        c.nid_celebracion,
        c.stitulo_anuncio,
        c.smensaje_anuncio,
        c.dfecha_inicio,
        c.dfecha_fin,
        c.ddatetime_creador,
        c.ddatetime_update,
        c.nusuario_creador,
        c.nusuario_update,
        c.sefecto_animacion,
        c.nestado
    FROM tenant0001_dbo.celebracion c
    WHERE c.dfecha_inicio <= par_current_date
      AND c.dfecha_fin >= par_current_date
    ORDER BY c.dfecha_inicio DESC;
END;
$$;
```