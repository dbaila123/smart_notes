```SQL
CREATE OR REPLACE FUNCTION fn_smart_get_celebraciones(
    p_month INTEGER,
    p_state_one TEXT,
    p_state_two TEXT,
    p_search_text TEXT,
    p_num INTEGER,
    p_size INTEGER,
    p_campo TEXT,
    p_order TEXT
)
RETURNS TABLE (
    spersona_nombre TEXT,
    dfecha_celebration DATE,
    dfecha_nacimiento DATE,
    dfecha_ingreso DATE,
    stelefono TEXT,
    ncodigo_tipo_modalidad INTEGER,
    surl_foto TEXT,
    stip_modalidad TEXT,
    ncodigo_sede INTEGER,
    ssede TEXT,
    ncodigo_estado INTEGER,
    sestado TEXT,
    nyears INTEGER,
    ntotal INTEGER
)
LANGUAGE plpgsql AS
$$
DECLARE
    view_sql TEXT := '
        SELECT 
            spersona_nombre, 
            dfecha_celebration::DATE, 
            dfecha_nacimiento::DATE, 
            dfecha_ingreso::DATE, 
            stelefono, 
            ncodigo_tipo_modalidad, 
            surl_foto, 
            stip_modalidad, 
            ncodigo_sede, 
            ssede, 
            ncodigo_estado, 
            sestado, 
            nyears 
        FROM v_celebraciones';
    where_clause TEXT;
    order_clause TEXT;
    pagination_clause TEXT;
    script TEXT;
    contador TEXT;
    total_count INTEGER;
BEGIN
    where_clause := format(' WHERE ((EXTRACT(MONTH FROM dfecha_nacimiento) = %s AND ncodigo_estado = 1) OR (EXTRACT(MONTH FROM dfecha_ingreso) = %s AND ncodigo_estado = 2))', p_month, p_month);
    IF p_state_one IS NOT NULL THEN
        where_clause := where_clause || format(' AND ncodigo_tipo_modalidad IN (%s)', p_state_one);
    END IF;
    
    IF p_state_two IS NOT NULL THEN
        where_clause := where_clause || format(' AND ncodigo_estado IN (%s)', p_state_two);
    END IF;

    IF p_search_text IS NOT NULL THEN
        where_clause := where_clause || format(' AND spersona_nombre ILIKE %L', '%' || p_search_text || '%');
    END IF;
    
    order_clause := format(' ORDER BY %I %s', p_campo, CASE WHEN LOWER(p_order) IN ('asc', 'desc') THEN p_order ELSE 'asc' END);
    pagination_clause := format(' OFFSET %s LIMIT %s', (p_num - 1) * p_size, p_size);
    contador := format('SELECT COUNT(*) FROM (%s %s) AS sub', view_sql, where_clause);
    EXECUTE contador INTO total_count;
    script := format( 'SELECT *, %s AS "ntotal" FROM (%s %s) AS sub %s %s', total_count, view_sql, where_clause, order_clause, pagination_clause);
    RETURN QUERY EXECUTE script;
END;
$$;