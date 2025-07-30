```sql
create function fn_get_modalidad_colaborador(p_collaborator_id integer, p_datetime timestamp)
    returns int
    language plpgsql
as
$$
declare
    var_modalidad_asistencia integer;
    var_estado_asistencia integer;
    var_tiene_remoto_por_horas boolean:=false;
    var_tiene_solicitud_por_dias boolean:=false;
begin
    SELECT
        nid_modalidad,
        nestado_asistencia
    INTO var_modalidad_asistencia, var_estado_asistencia
    FROM asistencias
    WHERE dfecha_asistencia = p_datetime::date
        and nid_colaborador = p_collaborator_id;

    if var_modalidad_asistencia <> 3 OR var_estado_asistencia IN (7,8,6,10,5) then
        SELECT EXISTS(
            SELECT 1
            FROM solicitudes s
                JOIN colaboradores c ON s.nid_colaborador = c.nid_colaborador
            WHERE c.nid_colaborador = p_collaborator_id
              AND s.dfecha_inicio = p_datetime::date
              AND s.nid_tipo_solicitud = 20
              AND p_datetime::time BETWEEN s.dHora_Inicio AND s.dHora_Fin
        ) INTO var_tiene_remoto_por_horas;

        SELECT EXISTS(
            SELECT 1
            FROM solicitudes s
                JOIN colaboradores c ON s.nid_colaborador = c.nid_colaborador
                JOIN tipos_solicitudes ts ON s.nid_tipo_solicitud  = ts.nid_tipo_solicitud
            WHERE c.nid_colaborador = p_collaborator_id
              AND p_datetime::date BETWEEN s.dfecha_inicio AND s.dfecha_fin
              AND ts.nafecta_asistencia = 1
              AND s.nestado_solicitud = 3
              AND ts.ntipo_frecuencia = 2
        ) INTO var_tiene_solicitud_por_dias;

        if var_tiene_remoto_por_horas OR var_tiene_solicitud_por_dias then
            return 3;
        end if;

    else
        return 3;
    end if;

    return var_modalidad_asistencia;
end;
$$;