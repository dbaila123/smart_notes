```sql
RETURNS TABLE (colaboradoresConcatenados TEXT) AS $$
BEGIN
    RETURN QUERY
    SELECT STRING_AGG(nid_colaborador::TEXT, ',' ORDER BY nid_colaborador) AS colaboradoresConcatenados
    FROM (
        SELECT DISTINCT cs.nid_colaborador
        FROM colaboradores_supervisor cs
        JOIN colaboradores colab ON colab.nid_colaborador = cs.nid_colaborador 
        LEFT JOIN colaboradores superv ON cs.nid_supervisor = superv.nid_colaborador
        WHERE cs.nestado = 1
            AND colab.nestado_colaborador = 1
            AND superv.nestado_colaborador = 1
            AND cs.nid_supervisor = p_nid_supervisor
        UNION
        SELECT DISTINCT superv.nid_colaborador
        FROM colaboradores superv
        WHERE superv.nestado_colaborador = 1
            AND superv.nid_colaborador = p_nid_supervisor
    ) AS combined_results;
END;
$$ LANGUAGE plpgsql;