```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_get_amount_left_to_be_paid(par_nid_solicitud INT)
RETURNS INT
LANGUAGE plpgsql
AS $$
DECLARE v_monto INT;
BEGIN
    SELECT nMonto_Por_Pagar
    INTO v_monto
    FROM tenant0001_dbo.tesoreria_cuotas_consolidado
    WHERE nid_solicitud_tesoreria = par_nid_solicitud;

    RETURN v_monto;
END;
$$;
```