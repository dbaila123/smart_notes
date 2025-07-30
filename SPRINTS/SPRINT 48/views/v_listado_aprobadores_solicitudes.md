```sql
create view v_listado_aprobadores_solicitudes as
    SELECT DISTINCT
        nid_solictud,
        snombre_aprobador,
        nid_aprobador,
        dfecha_solicitud,
        dfecha_inicio,
        dfecha_fin
    FROM (
            SELECT
                solapro.nid_solictud,
                (
                    SELECT papro.spersona_nombre
                    FROM personas papro
                    JOIN usuarios uapro
                        on (jsondata.value->>'id')::integer = uapro.nid_usuario
                        and papro.nid_persona = uapro.nid_persona
                ) AS snombre_aprobador,
                (jsondata.value->>'id')::int AS nid_aprobador,
                solapro.dfecha_solicitud,
                solapro.dfecha_inicio,
                solapro.dfecha_fin
            FROM solicitudes solapro
            JOIN LATERAL (
                SELECT
                    jsonb_array_elements(
                        CASE
                            WHEN solApro.saprobadores IN ('[]', 'END', '') OR solApro.saprobadores IS NULL THEN
                                '[{"id":0,"name":"NULL"}]'::jsonb
                            ELSE
                                solApro.saprobadores::jsonb
                        END
                    ) AS value
                ) AS jsondata ON TRUE
            WHERE solApro.saprobadores NOT IN ('[]', 'END')
              AND solApro.saprobadores IS NOT NULL
              AND solApro.nestado_solicitud != 6
              AND solApro.nid_tipo_solicitud != 9

            UNION ALL

            SELECT
                sol.nid_solictud,
                p.spersona_nombre AS sNombre_Aprobador,
                u.nid_usuario AS nId_Aprobador,
                sol.dfecha_solicitud,
                sol.dfecha_inicio,
                sol.dfecha_fin
            FROM solicitudes sol
                     JOIN usuarios u ON sol.nusuario_update = u.nid_usuario
                     JOIN personas p ON p.nid_persona = u.nid_persona
            WHERE sol.nid_tipo_solicitud != 9
              AND sol.nestado_solicitud != 6

         ) AS combinedResults;
