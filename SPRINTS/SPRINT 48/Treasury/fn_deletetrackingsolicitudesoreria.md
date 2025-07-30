```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_delete_tracking_solicitud_tesoreria(
    par_id integer
) RETURNS void
LANGUAGE plpgsql
AS $$
BEGIN
    DELETE FROM notificaciones
    WHERE nid_entidad = par_id;

    DELETE FROM solicitud_tesoreria_historico
    WHERE nid_solicitud_tesoreria = par_id;

    DELETE FROM aprobadores
    WHERE nid_entidad = par_id;

    DELETE FROM solicitud_tesoreria
    WHERE nid_solicitud_tesoreria = par_id;
END;
$$;
```