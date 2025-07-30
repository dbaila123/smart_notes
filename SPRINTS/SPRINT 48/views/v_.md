```sql
create view v_listado_emails_personas(scodigo, sdescripcion, stipo_configuracion) as
SELECT concat('2-', p.nid_persona)                                                 AS scodigo,
       concat(p.sprimer_nombre, ' ', p.sape_paterno, ' (Laboral: ', u.semail, ')') AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text                                                    AS stipo_configuracion
FROM usuarios u
         JOIN personas p ON u.nid_persona = p.nid_persona
WHERE u.nestado = 1
UNION
SELECT concat('1-', p.nid_persona)                                                  AS scodigo,
       concat(p.sprimer_nombre, ' ', p.sape_paterno, ' (Personal: ', e.semail, ')') AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text                                                     AS stipo_configuracion
FROM personas p
         JOIN emails e ON e.nid_persona = p.nid_persona
         JOIN usuarios u ON u.nid_persona = p.nid_persona
         JOIN colaboradores c ON c.nid_persona = p.nid_persona
WHERE p.nestado = 1
  AND u.nestado = 1
  AND c.nestado_colaborador = 1
  AND e.nprincipal = 0
UNION
SELECT concat('3-', eu.nid_empresa)                       AS scodigo,
       concat(p.spersona_nombre, ' - Correos personales') AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text                           AS stipo_configuracion
FROM empresas_usuarias eu
         JOIN personas p ON p.nid_persona = eu.nid_persona
WHERE eu.nestado = 1
UNION
SELECT concat('4-', eu.nid_empresa)                      AS scodigo,
       concat(p.spersona_nombre, ' - Correos laborales') AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text                          AS stipo_configuracion
FROM empresas_usuarias eu
         JOIN personas p ON p.nid_persona = eu.nid_persona
WHERE eu.nestado = 1
UNION
SELECT concat('5-', vsa.nid_supervisor)                          AS scodigo,
       concat('EQUIPO: ', p.sprimer_nombre, ' ', p.sape_paterno) AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text                                  AS stipo_configuracion
FROM v_supervisores_asginados vsa
         JOIN colaboradores c ON c.nid_colaborador = vsa.nid_supervisor
         JOIN personas p ON c.nid_persona = p.nid_persona
WHERE c.nestado_colaborador = 1
UNION
SELECT '6'::text                      AS scodigo,
       'TODOS LOS SUPERVISORES'::text AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text       AS stipo_configuracion
UNION
SELECT concat('7-', s.nid_sede)                                 AS scodigo,
       concat('Sede: ', s.sdescripcion, '- Correos Personales') AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text                                 AS stipo_configuracion
FROM sedes s
WHERE s.nestado = 1
  AND lower(s.sdescripcion) !~~ lower('%principal%'::text)
UNION
SELECT concat('8-', s.nid_sede)                                AS scodigo,
       concat('Sede: ', s.sdescripcion, '- Correos Laborales') AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text                                AS stipo_configuracion
FROM sedes s
WHERE s.nestado = 1
  AND lower(s.sdescripcion) !~~ lower('%principal%'::text)
UNION
SELECT '9'::text                                     AS scodigo,
       'TODO EL PERSONAL - Correos Personales'::text AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text                      AS stipo_configuracion
UNION
SELECT '10'::text                                   AS scodigo,
       'TODO EL PERSONAL - Correos Laborales'::text AS sdescripcion,
       'TIPO-EMAIL-GROUP'::text                     AS stipo_configuracion;

alter table v_listado_emails_personas
    owner to postgres;

