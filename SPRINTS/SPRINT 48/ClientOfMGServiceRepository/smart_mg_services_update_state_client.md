```sql
CREATE OR REPLACE FUNCTION fn_smart_mg_services_update_state_client(
    par_nid_cliente INT,
    par_nestado INT,
    par_ddatetime_update TIMESTAMP,
    par_nusuario_update INT
) RETURNS INTEGER
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE clientes
    SET nEstado = par_nestado,
        dDatetime_Update = par_ddatetime_update,
        nUsuario_Update = par_nusuario_update
    WHERE nId_Cliente = par_nid_cliente;
    IF FOUND THEN
        RETURN 1; -- Ok
    ELSE
        RETURN 0; -- Ninguna fila afectada
    END IF;
END;
$$;