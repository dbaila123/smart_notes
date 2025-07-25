```SQL
 -- listado de mailing (nombres de correos):
if 'mailing_list' = any(var_configuration_request) then
        RETURN QUERY
SELECT
    m.nid_mailingtemplate,
    m.sname,
    'MAILING_LIST'
FROM mailingtemplate m;
end if;

if 'rango_marcas' = any(var_configuration_request) then
        RETURN QUERY
SELECT
    c.scodigo::text,
    c.sdescripcion,
    'RANGO_MARCAS'
FROM configs c
WHERE stabla = 'RANGO_MARCAS';
end if;

if 'evaluation_template' = any(var_configuration_request) then
        RETURN QUERY
SELECT
    f.nid_form_template::text,
    f.sname,
    'EVALUATION_TEMPLATE'
FROM form_template f;
end if;

if 'evaluation' = any(var_configuration_request) then
        RETURN QUERY
SELECT
    e.nid_evaluation::text,
    e.sname,
    'EVALUATION'
FROM evaluations e;
end if;

if 'form_state' = any(var_configuration_request) then
        RETURN QUERY
SELECT
    c.scodigo::text,
    c.sdescripcion,
    'FORM_STATE'
FROM configs c
WHERE stabla like('ESTADO_FORMULARIO');
end if;

if 'my_evaluations' = any(var_configuration_request) then
        RETURN QUERY
SELECT
                e.nid_evaluation::text,
                e.sname,
                'MY_EVALUATIONS'
            FROM (
                     SELECT DISTINCT
                         e.nid_evaluation,
                         e.sname,
                         CASE
                             WHEN e.nstate = 1 THEN 1
                             WHEN (CURRENT_TIMESTAMP - INTERVAL  '5 hours')::date BETWEEN e.dinit_date AND e.dend_date THEN 2
                             WHEN (CURRENT_TIMESTAMP - INTERVAL '5 hours')::date < e.dinit_date THEN 4
                             WHEN (CURRENT_TIMESTAMP - INTERVAL '5 hours')::date > e.dend_date THEN 3
                             END AS nstate -- cuando se filtre por estados
                     FROM evaluations e
                              JOIN forms f on e.nid_evaluation = f.nid_evaluation
                              JOIN Evaluators ev on f.nid_form = ev.nid_form
                     WHERE ev.nid_evaluator = 6

                 ) e
            WHERE e.nstate NOT IN (1, 4)
            ORDER BY e.nid_evaluation DESC;
end if;

if 'forms' = any(var_configuration_request) then
        RETURN QUERY
SELECT ft.nid_form_type::text,
       ft.sname,
       'FORMS'
FROM form_type ft
WHERE ft.nid_form_type NOT IN (
    SELECT distinct nid_formtype_padre
    FROM form_type ft2
    WHERE ft2.nid_formtype_padre IS NOT NULL)
  AND ft.bstate = 1;
end if;

if 'my_evaluations_completed' = any(var_configuration_request) then
        RETURN QUERY
SELECT
    e.nid_evaluation::text,
    e.sname,
    'MY_EVALUATIONS_COMPLETED'
FROM (
         SELECT DISTINCT
             e.nid_evaluation,
             e.sname,
             CASE
                 WHEN e.nstate = 1 THEN 1
                 WHEN (CURRENT_TIMESTAMP - INTERVAL '5 hours') BETWEEN e.dinit_date AND e.dend_date THEN 2
                 WHEN (CURRENT_TIMESTAMP - INTERVAL '5 hours') < e.dinit_date THEN 4
                 WHEN (CURRENT_TIMESTAMP - INTERVAL '5 hours') > e.dend_date THEN 3
             END AS nstate, -- Cuando se filtre por estados
             e.dshare_results
         FROM evaluations e
                  JOIN forms f ON e.nid_evaluation = f.nid_evaluation
                  JOIN evaluators ev ON f.nid_form = ev.nid_form
         WHERE ev.nid_evaluator = p_collaborator_id
     ) e
WHERE e.nstate = 3
  AND e.dshare_results IS NOT NULL
  AND e.dshare_results <= (CURRENT_TIMESTAMP - INTERVAL '5 hours')
ORDER BY e.nid_evaluation DESC;
end if;

if 'planned_closing' = any(var_configuration_request) then
        RETURN QUERY
SELECT
    car.nid_cierre::text,
    car.dfecha_cierre_planificado::text,
    'PLANNED_CLOSING'
FROM cierreasistenciareportes car
WHERE car.dfecha_cierre_planificado is not null
order by car.nid_cierre desc
LIMIT 1;
end if;

if 'state_dashboard' = any(var_configuration_request) then
        RETURN QUERY
SELECT
    c.scodigo::text,
    c.sdescripcion,
    'ESTADO_INDICADORES'
FROM configs c
WHERE stabla = 'ESTADO_INDICADORES'
AND nestado = 1;
end if;