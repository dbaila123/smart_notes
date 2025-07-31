```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_smart_get_contratos(
	snombre_colaborador text, 
	fechaini text, 
	fechafin text, 
	sfilterone text, 
	sfiltertwo text, 
	sfilterthree text, 
	pnum integer, 
	size integer, 
	campo text, 
	orden text
)
RETURNS TABLE(
	nid integer, 
	snombre_colaborador_out text, 
	dfecha_ini date, 
	dfecha_fin date, 
	ntipo_modalidad integer, 
	stipo_modalidad text, 
	nid_sede integer, 
	snombre_sede text, 
	nid_cargo integer, 
	snombre_cargo text, 
	nid_empresa integer, 
	snombre_empresa text, 
	dfec_ingreso date, 
	nestado integer, 
	snombre_estado text, 
	surl_foto text, 
	nestado_colaborador integer, 
	sestado_colaborador text, 
	surl_file text, 
	bhistorico boolean, 
	sjson_contrato_old text, 
	total integer
)
LANGUAGE plpgsql
AS $$
DECLARE
    where_clauses text := '';
    order_clause text := '';
    pagination_clause text := '';
    total_count integer := 0;
    base_query text := 'SELECT * FROM tenant0001_dbo.v_listado_contratos';
BEGIN
    IF snombre_colaborador IS NOT NULL THEN
        where_clauses := where_clauses || 'sNombre_Colaborador ILIKE ''%' || replace(snombre_colaborador, '''', '''''') || '%'' AND ';
    END IF;

    IF fechaini IS NOT NULL AND fechafin IS NOT NULL THEN
        where_clauses := where_clauses || 'dFec_Ingreso BETWEEN ''' || fechaini || ''' AND ''' || fechafin || ''' AND ';
    END IF;

    IF sfilterone IS NOT NULL THEN
        where_clauses := where_clauses || 'nTipo_Modalidad IN (' || sfilterone || ') AND ';
    END IF;

    IF sfiltertwo IS NOT NULL THEN
        where_clauses := where_clauses || 'nEstado IN (' || sfiltertwo || ') AND ';
    END IF;

    IF sfilterthree IS NOT NULL THEN
        where_clauses := where_clauses || 'nEstado_Colaborador IN (' || sfilterthree || ') AND ';
    END IF;

    IF length(where_clauses) > 0 THEN
        where_clauses := ' WHERE ' || left(where_clauses, length(where_clauses) - 5);
    END IF;

    IF campo IS NOT NULL AND orden IS NOT NULL THEN
        order_clause := ' ORDER BY ' || campo || ' ' || orden;
    ELSE
        order_clause := ' ORDER BY dFec_Ingreso DESC';
    END IF;

    IF pnum IS NOT NULL AND size IS NOT NULL THEN
        pagination_clause := ' LIMIT ' || size || ' OFFSET ' || ((pnum - 1) * size);
    END IF;

    EXECUTE format('SELECT count(*) FROM (%s %s) AS sub', base_query, where_clauses) INTO total_count;

    RETURN QUERY EXECUTE format(
        'SELECT *, %s AS total FROM (%s %s) AS sub %s %s',
        total_count, base_query, where_clauses, order_clause, pagination_clause
    );
END;
$$;
```