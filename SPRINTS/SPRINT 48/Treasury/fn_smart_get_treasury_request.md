```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_smart_get_treasury_request(
	par_nid_user integer,
	par_snombrecolaborador character varying, 
	par_sslugtogetlist character varying, 
	par_fechaini character varying, 
	par_fechafin character varying, 
	par_fechaefectoinicio character varying, 
	par_fechaefectofin character varying, 
	par_sfilterone character varying, 
	par_sfiltertwo character varying, 
	par_sfilterthree character varying, 
	par_sfilterfour character varying, 
	par_sfilterfive character varying, 
	par_sfilternotification character varying, 
	par_pnum integer, 
	par_size integer, 
	par_campo character varying, 
	par_order character varying)
 RETURNS TABLE(
	 nid_solicitud_tesoreria integer, 
	 nid_colaborador integer, 
	 surl_foto text, 
	 snombre_colaborador text, 
	 dfecha_efecto timestamp without time zone, 
	 dfecha_creacion timestamp without time zone, 
	 nmonto numeric, 
	 smotivo text, 
	 ntipo_moneda integer, 
	 stipo_moneda text, 
	 nid_tipo_solicitud integer, 
	 stipo_solicitud text, 
	 nid_categoria integer, 
	 snombre_usuario_update text, 
	 sobservacion text, 
	 nestado_solicitud integer, 
	 sestado_solicitud text, 
	 nid_proyecto integer, 
	 snombre_proyecto text, 
	 nid_requerimiento integer, 
	 snombre_requerimiento text, 
	 nid_lider_pry integer, 
	 nid_director_pry integer, 
	 ncuotas_pagadas integer, 
	 nmonto_por_pagar numeric, 
	 nmonto_depositado numeric, 
	 ncodigo_moneda_reembolso text, 
	 stipo_moneda_reembolso text, 
	 ntotal_cuotas integer, 
	 surl_files_on_creation text, 
	 surl_files_on_refund text, 
	 proyectos json, ntotal integer
)
LANGUAGE plpgsql
AS $$
DECLARE
    var_script TEXT;
    var_viewclause TEXT := '';
    var_whereclause TEXT := '';
    var_orderclause TEXT := '';
    var_paginationclause TEXT := '';
    var_contador TEXT;
    var_nid_colaborador INTEGER;
    var_total INTEGER := 0;
    var_condition TEXT;
    var_conditions TEXT[] := '{}';
BEGIN
    IF par_nid_user IS NULL OR par_sslugtogetlist IS NULL THEN
        RAISE EXCEPTION 'Los parámetros par_nid_user y par_sslugtogetlist son requeridos';
    END IF;

    BEGIN
        SELECT c.nid_colaborador INTO var_nid_colaborador
        FROM usuarios u
        JOIN personas p ON u.nid_persona = p.nid_persona
        JOIN colaboradores c ON p.nid_persona = c.nid_persona
        WHERE u.nid_usuario = par_nid_user;
        
        IF var_nid_colaborador IS NULL THEN
            RAISE EXCEPTION 'No se encontró colaborador para el usuario %', par_nid_user;
        END IF;
    EXCEPTION WHEN OTHERS THEN
        RAISE EXCEPTION 'Error al obtener ID de colaborador: %', SQLERRM;
    END;

    IF par_sslugtogetlist = 'TESORERIA-LISTA-INDIVIDUAL' THEN
        var_viewclause := 'SELECT * FROM v_listado_tesoreria WHERE nid_colaborador = ' || var_nid_colaborador;
    ELSIF par_sslugtogetlist = 'TESORERIA-LISTADO' THEN
        var_viewclause := 'SELECT * FROM v_listado_tesoreria';
    ELSE
        RAISE EXCEPTION 'Valor no válido para par_sslugtogetlist: %', par_sslugtogetlist;
    END IF;

    IF par_snombrecolaborador IS NOT NULL AND par_snombrecolaborador != '' THEN
        var_condition := tenant0001_dbo.fn_get_like_condition('snombre_colaborador', par_snombrecolaborador);
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF par_fechaini IS NOT NULL AND par_fechaini != '' AND par_fechafin IS NOT NULL AND par_fechafin != '' THEN
        var_condition := tenant0001_dbo.fn_get_dates_condition('dfecha_creacion', par_fechaini, par_fechafin);
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF par_fechaefectoinicio IS NOT NULL AND par_fechaefectoinicio != '' AND par_fechaefectofin IS NOT NULL AND par_fechaefectofin != '' THEN
        var_condition := tenant0001_dbo.fn_get_dates_condition('dfecha_efecto', par_fechaefectoinicio, par_fechaefectofin);
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF par_sfilterone IS NOT NULL AND par_sfilterone != '' THEN
        var_condition := 'nid_categoria IN (' || par_sfilterone || ')';
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF par_sfiltertwo IS NOT NULL AND par_sfiltertwo != '' THEN
        var_condition := 'nid_tipo_solicitud IN (' || par_sfiltertwo || ')';
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF par_sfilterthree IS NOT NULL AND par_sfilterthree != '' THEN
        var_condition := 'nestado_solicitud IN (' || par_sfilterthree || ')';
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF par_sfilterfive IS NOT NULL AND par_sfilterfive != '' THEN
        var_condition := 'ntipo_moneda IN (' || par_sfilterfive || ')';
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF par_sfilternotification IS NOT NULL AND par_sfilternotification != '' THEN
        var_condition := 'nid_solicitud_tesoreria IN (' || par_sfilternotification || ')';
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF array_length(var_conditions, 1) > 0 THEN
        var_whereclause := ' WHERE ' || array_to_string(var_conditions, ' AND ');
    END IF;

    var_orderclause := tenant0001_dbo.fn_get_order_clause(par_campo, par_order, 'dfecha_creacion');
    var_paginationclause := tenant0001_dbo.fn_get_pagination_clause(par_pnum, par_size);

    BEGIN
        var_contador := 'SELECT count(*) FROM (' || var_viewclause || var_whereclause || ') t';
        EXECUTE var_contador INTO var_total;
    EXCEPTION WHEN OTHERS THEN
        RAISE EXCEPTION 'Error al contar registros: %', SQLERRM;
    END;

    var_script := 'SELECT t.*, ' || var_total || ' AS ntotal FROM (' || var_viewclause || var_whereclause || ') t ' || 
                 var_orderclause || ' ' || var_paginationclause;
    
    IF var_script IS NULL OR trim(var_script) = '' THEN
        RAISE EXCEPTION 'La consulta generada está vacía';
    END IF;

    BEGIN
        RETURN QUERY EXECUTE var_script;
    EXCEPTION WHEN OTHERS THEN
        RAISE EXCEPTION 'Error al ejecutar consulta: %. Consulta: %', SQLERRM, var_script;
    END;
END;
$$;

```