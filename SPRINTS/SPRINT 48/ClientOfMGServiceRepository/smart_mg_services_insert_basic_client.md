```sql
CREATE OR REPLACE FUNCTION fn_smart_mg_services_insert_basic_client(
	-- p_ = parámetro
    p_nid_persona INTEGER DEFAULT NULL,
    p_ddatetime_creador TIMESTAMP DEFAULT NULL,
    p_ddatetime_delete TIMESTAMP DEFAULT NULL,
    p_ddatetime_update TIMESTAMP DEFAULT NULL,
    p_nestado INTEGER DEFAULT NULL,
    p_nusuario_creador INTEGER DEFAULT NULL,
    p_nusuario_delete INTEGER DEFAULT NULL,
    p_nusuario_update INTEGER DEFAULT NULL,
    p_sestado_cliente VARCHAR(50) DEFAULT NULL
)
RETURNS INTEGER AS
$$
DECLARE
    v_nid_cliente INTEGER := 0;
BEGIN
    IF p_nid_persona IS NOT NULL AND p_nusuario_creador IS NOT NULL THEN
        INSERT INTO clientes (
            nid_persona,
            ddatetime_creador,
            ddatetime_delete,
            ddatetime_update,
            nestado,
            nusuario_creador,
            nusuario_delete,
            nusuario_update,
            sestado_cliente
        ) VALUES (
            p_nid_persona,
            p_ddatetime_creador,
            p_ddatetime_delete,
            p_ddatetime_update,
            p_nestado,
            p_nusuario_creador,
            p_nusuario_delete,
            p_nusuario_update,
            p_sestado_cliente
        )
        RETURNING nid_cliente INTO v_nid_cliente;
        RETURN v_nid_cliente;
    ELSE
        RETURN 0;
    END IF;
END;
$$ LANGUAGE plpgsql;