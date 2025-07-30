```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_smart_get_treasury_request_footer(
    par_nid_user INTEGER,
    par_snombrecolaborador VARCHAR,
    par_sslugtogetlist VARCHAR,
    par_fechaini VARCHAR,
    par_fechafin VARCHAR,
    par_fechaefectoinicio VARCHAR,
    par_fechaefectofin VARCHAR,
    par_sfilterone VARCHAR,
    par_sfiltertwo VARCHAR,
    par_sfilterthree VARCHAR,
    par_sfilterfour VARCHAR,
    par_sfilterfive VARCHAR,
    par_sfilternotification VARCHAR
)
RETURNS TABLE (
    ntotal_gastos_pendientes TEXT,
    ntotal_gastos_reembolsados TEXT,
    ntotal_prestamos_pendientes NUMERIC,
    ntotal_prestamos_proceso NUMERIC
)
LANGUAGE plpgsql
AS $function$
DECLARE
    var_gastosp TEXT;
    var_gastosr TEXT;
    var_prestamosp TEXT;
    var_prestamospr TEXT;
    var_viewclause TEXT := '';
    var_whereclause TEXT := '';
    var_nid_colaborador INTEGER;
    var_nullvalues TEXT := '0,00';
    var_condition TEXT;
    var_conditions TEXT[] := '{}';
    var_final_query TEXT;
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

    BEGIN
        IF par_sfilterfive IS NOT NULL THEN
            SELECT string_agg(concat(c.sicono, ' 0,00'), ' - ') INTO var_nullvalues
            FROM configs c
            WHERE c.stabla = 'TIPO_MONEDA' AND c.nestado = 1 
            AND c.scodigo = ANY(string_to_array(par_sfilterfive, ','));
        ELSE
            SELECT string_agg(concat(c.sicono, ' 0,00'), ' - ') INTO var_nullvalues
            FROM configs c
            WHERE c.stabla = 'TIPO_MONEDA' AND c.nestado = 1;
        END IF;
    EXCEPTION WHEN OTHERS THEN
        var_nullvalues := '0,00';
    END;

    CASE par_sslugtogetlist
        WHEN 'TESORERIA-LISTA-INDIVIDUAL' THEN
            var_viewclause := 'SELECT lt.*, c.sicono 
                              FROM v_listado_tesoreria lt
                              LEFT JOIN configs c ON c.scodigo = lt.ntipo_moneda::varchar 
                              AND c.stabla = ''TIPO_MONEDA'' AND c.nestado = 1
                              WHERE lt.nid_colaborador = ' || var_nid_colaborador;
        
        WHEN 'TESORERIA-LISTA-EQUIPO' THEN
            IF par_sfilterfour = '0' THEN
                var_viewclause := 'SELECT * FROM v_listado_tesoreria 
                                  WHERE nid_lider = ' || var_nid_colaborador || 
                                  ' OR nid_director = ' || var_nid_colaborador;
            ELSIF par_sfilterfour = '1' THEN
                var_viewclause := 'SELECT * FROM v_listado_tesoreria 
                                  WHERE nid_lider_pry = ' || var_nid_colaborador || 
                                  ' OR nid_director_pry = ' || var_nid_colaborador;
            ELSE
                var_viewclause := 'SELECT * FROM v_listado_tesoreria 
                                  WHERE nid_lider = ' || var_nid_colaborador || 
                                  ' OR nid_director = ' || var_nid_colaborador || 
                                  ' OR nid_lider_pry = ' || var_nid_colaborador || 
                                  ' OR nid_director_pry = ' || var_nid_colaborador;
            END IF;
        
        WHEN 'TESORERIA-LISTADO' THEN
            var_viewclause := 'SELECT lt.*, c.sicono 
                              FROM v_listado_tesoreria lt
                              LEFT JOIN configs c ON c.scodigo = lt.ntipo_moneda::varchar 
                              AND c.stabla = ''TIPO_MONEDA'' AND c.nestado = 1';
        
        ELSE
            RAISE EXCEPTION 'Valor no válido para par_sslugtogetlist: %', par_sslugtogetlist;
    END CASE;

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
        var_condition := 'ntipo_moneda::integer IN (' || par_sfilterfive || ')';
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF par_sfilternotification IS NOT NULL AND par_sfilternotification != '' THEN
        var_condition := 'nid_solicitud_tesoreria IN (' || par_sfilternotification || ')';
        var_conditions := array_append(var_conditions, var_condition);
    END IF;

    IF array_length(var_conditions, 1) > 0 THEN
        var_whereclause := ' WHERE ' || array_to_string(var_conditions, ' AND ');
    END IF;

    IF var_viewclause IS NULL OR trim(var_viewclause) = '' THEN
        RAISE EXCEPTION 'La consulta base no puede estar vacía';
    END IF;

    var_gastosp := 'WITH tabla AS (
                    SELECT a.sicono || '' '' || COALESCE(SUM(a.nmonto), 0)::numeric(18,2) AS total, 
                           a.ntipo_moneda
                    FROM (' || var_viewclause || var_whereclause || ' AND nid_categoria = 1 AND nestado_solicitud = 1) a
                    GROUP BY a.ntipo_moneda, a.sicono
                   )
                   SELECT COALESCE(string_agg(REPLACE(total, ''.'', '',''), '' - '' ORDER BY ntipo_moneda), ''' || var_nullvalues || ''')
                   FROM tabla';

    var_gastosr := 'WITH tabla AS (
                    SELECT a.sicono || '' '' || COALESCE(SUM(a.nmonto), 0)::numeric(18,2) AS total, 
                           a.ntipo_moneda
                    FROM (' || var_viewclause || var_whereclause || ' AND nid_categoria = 1 AND nestado_solicitud = 3) a
                    GROUP BY a.ntipo_moneda, a.sicono
                   )
                   SELECT COALESCE(string_agg(REPLACE(total, ''.'', '',''), '' - '' ORDER BY ntipo_moneda), ''' || var_nullvalues || ''')
                   FROM tabla';

    var_prestamosp := 'SELECT COALESCE(SUM(nmonto), 0)::numeric(18,2) AS ntotal_prestamos_pendientes
                       FROM (' || var_viewclause || var_whereclause || ' AND nid_categoria = 2 AND nestado_solicitud = 1) a';

    var_prestamospr := 'SELECT COALESCE(SUM(nmonto), 0)::numeric(18,2) AS ntotal_prestamos_proceso
                        FROM (' || var_viewclause || var_whereclause || ' AND nid_categoria = 2 AND nestado_solicitud = 6) a';

    var_final_query := 'SELECT 
             (' || COALESCE(var_gastosp, 'SELECT ''' || var_nullvalues || '''') || ')::text AS ntotal_gastos_pendientes,
             (' || COALESCE(var_gastosr, 'SELECT ''' || var_nullvalues || '''') || ')::text AS ntotal_gastos_reembolsados,
             (' || COALESCE(var_prestamosp, 'SELECT 0') || ')::numeric AS ntotal_prestamos_pendientes,
             (' || COALESCE(var_prestamospr, 'SELECT 0') || ')::numeric AS ntotal_prestamos_proceso';

    IF var_final_query IS NULL OR trim(var_final_query) = '' THEN
        RAISE EXCEPTION 'La consulta final no puede estar vacía';
    END IF;

    BEGIN
        RETURN QUERY EXECUTE var_final_query;
    EXCEPTION WHEN OTHERS THEN
        RAISE EXCEPTION 'Error al ejecutar consulta final: %. Consulta: %', SQLERRM, var_final_query;
    END;
END;
$$;

```