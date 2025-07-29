```sql
create or replace view v_supervisores_asginados(nid_supervisor, spersona_nombre, nestado) as
SELECT DISTINCT cs.nid_supervisor,
                p.spersona_nombre,
                c.nestado_colaborador AS nestado
FROM colaboradores_supervisor cs
JOIN colaboradores c ON cs.nid_supervisor = c.nid_colaborador
JOIN personas p ON p.nid_persona = c.nid_persona
WHERE p.spersona_nombre NOT ILIKE '%smart%'
  AND c.nestado_colaborador = 1;