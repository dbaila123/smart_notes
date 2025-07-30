```sql
create or replace funcion fn_get_configurations(p_configurations_string text, p_user_id integer, p_project_id integer, p_requirement_id integer, p_slug_listado text, p_collaborator_id integer DEFAULT 0)
    returns TABLE(scodigo text, sdescription text, stipo_configuration text)
    language plpgsql
as
$$
declare
    var_configuration_request text[];
    var_modalidad integer;
begin
    if p_collaborator_id = 0 then
        p_collaborator_id := (select function_IdUserORIdColabordor('usuario', p_user_id));
    end if;

    var_configuration_request := string_to_array(p_configurations_string, '.');

    if 'coordinadores_supervisor' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_coordinador::text,
            p.spersona_nombre,
            'COORDINADORES_SUPERVISOR'
        FROM coordinadores c
        JOIN coordinadores_proyectos cp on cp.nid_coordinador = c.nid_coordinador
        JOIN personas p on c.nid_persona = p.nid_persona
        JOIN supervisor_proyecto sp on sp.nid_proyecto = cp.nid_proyecto
        JOIN coordinadores supervisor on supervisor.nid_coordinador = sp.nid_supervisor
        JOIN personas p_sup on p_sup.nid_persona = supervisor.nid_persona
        JOIN usuarios u_sup on u_sup.nid_persona = p_sup.nid_persona
        WHERE u_sup.nid_usuario = p_user_id
          AND c.nEstado = 1;
    end if;

    if 'coordinador' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_coordinador::text,
            p.spersona_nombre,
            'COORDINADOR'
        FROM coordinadores c
        JOIN personas p on c.nid_persona = p.nid_persona
        WHERE c.nestado = 1;
    end if;

    if 'coordinador_all' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_coordinador::text,
            p.spersona_nombre,
            'COORDINADOR_ALL'
        FROM coordinadores c
        LEFT JOIN personas p on c.nid_persona = p.nid_persona;
    end if;

    if 'coordinador_suggest' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_coordinador::text,
            p.spersona_nombre,
            'COORDINADOR_SUGGEST'
        FROM coordinadores c
        JOIN personas p on c.nid_persona = p.nid_persona;
    end if;

    if 'especialidades' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            e.nid_especialidad::text,
            e.sdescripcion,
            'ESPECIALIDADES'
        FROM especialidades e
        WHERE e.nestado = 1;
    end if;

    if 'ocupaciones' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            nid_ocupacion::text,
            sdescripcion,
            'OCUPACIONES'
        FROM ocupaciones
        WHERE nestado = 1;
    end if;

    if 'nacionalidades' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            nid_pais::text,
            sgentilicio,
            'NACIONALIDADES'
        FROM paises
        WHERE nestado = 1
        AND sgentilicio is not null;
    end if;

    if 'estados_civil' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
               c.scodigo,
                c.sdescripcion,
                'ESTADOS_CIVIL'
            FROM configs c
            WHERE c.nestado = 1
              AND c.stabla = 'ESTADO_CIVIL';
    end if;

    if 'fecha_min_reactivar_colaborador' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.scodigo,
            c.sdescripcion,
            'FECHA_REACTIVAR_COLABORADOR'
        FROM configs c
        WHERE c.stabla = 'FECHA_MIN_REACTIVAR_COLAB'
        AND c.nestado = 1;
    end if;

    if 'generos' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.scodigo,
            c.sdescripcion,
            'GENEROS'
        FROM configs c
        WHERE stabla = 'GENERO'
        AND nestado = 1;
    end if;

    if 'tipos_persona' =  any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.scodigo,
            c.sdescripcion,
            'TIPOS_PERSONA'
        FROM configs c
        WHERE stabla = 'TIPO_PERSONA'
        AND nestado = 1;
    end if;

    if 'areas' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            a.nid_area::text,
            a.sdescripcion,
            'AREAS'
        FROM areas a
        WHERE nestado = 1;
    end if;

    if 'empresas' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            eu.nid_empresa::text,
            p.spersona_nombre,
            'EMPRESAS'
        FROM empresas_usuarias eu
        LEFT JOIN personas p on eu.nid_persona = p.nid_persona;
    end if;

    if 'tipos_colaborador' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.scodigo,
            c.sdescripcion,
            'TIPOS_COLABORADOR'
        FROM configs c
        WHERE c.stabla = 'TIPO_COLABORADOR'
        AND c.nestado = 1;
    end if;

    if 'tipos_contratacion' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.scodigo,
            c.sdescripcion,
            'TIPOS_CONTRATACION'
        FROM configs c
        WHERE c.stabla = 'TIPO_CONTRATACION'
        AND c.nestado = 1;
    end if;

    if 'tipos_modalidad' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.scodigo,
            c.sdescripcion,
            'TIPOS_MODALIDAD'
        FROM configs c
        WHERE c.stabla = 'TIPO_MODALIDAD'
        AND c.nestado = 1;
    end if;

    if 'estados_colaborador' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.scodigo,
            c.sdescripcion,
            'ESTADOS_COLABORADOR'
        FROM configs c
        WHERE c.stabla = 'ESTADO_COLABORADOR'
        AND c.nestado = 1;
    end if;

    if 'cargos' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_cargo::text,
            c.sdescripcion,
            'CARGOS'
        FROM cargos c
        WHERE c.nestado_cargos = 1;
    end if;

    if 'lideres_directores' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_colaborador::text,
            p.spersona_nombre,
            CASE
                WHEN r.sdescripcion like '%LÍDER%' OR  r.sdescripcion = 'COORDINADOR' THEN 'LIDER'
                WHEN r.sdescripcion like '%DIRECTOR%' THEN 'DIRECTOR'
            END AS sTipo_Configuracion
        FROM colaboradores c
        JOIN personas p on c.nid_persona = p.nid_persona
        JOIN usuarios u on p.nid_persona = u.nid_persona
        JOIN roles_usuarios ru on u.nid_usuario = ru.nid_usuario
        JOIN roles r on ru.nid_rol = r.nid_rol
        WHERE r.sdescripcion like '%LÍDER%'
        OR r.sdescripcion like '%DIRECTOR%'
        OR r.sdescripcion = 'COORDINADOR';
    end if;

    if 'lideres_directores_all' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_colaborador::text,
            p.spersona_nombre,
            'LIDERES_DIRECTORES'
        FROM colaboradores c
        JOIN personas p on c.nid_persona = p.nid_persona
        JOIN usuarios u on p.nid_persona = u.nid_persona
        JOIN roles_usuarios ru on u.nid_usuario = ru.nid_usuario
        JOIN roles r on ru.nid_rol = r.nid_rol
        WHERE c.nestado_colaborador = 1
        AND r.sdescripcion like '%LÍDER%'
           OR r.sdescripcion like '%DIRECTOR%'
           OR r.sdescripcion = 'COORDINADOR';
    end if;

    if 'tipos_autenticacion' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.scodigo,
            c.sdescripcion,
            'TIPOS_AUTENTICACION'
        FROM configs c
        WHERE c.stabla = 'TIPO_AUTENTICACION'
          AND c.nestado = 1;
    end if;

    if 'tipos_documento' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'TIPOS_DOCUMENTO'
            FROM configs c
            WHERE c.stabla = 'TIPO_DOCUMENTO'
              AND c.nestado = 1;
    end if;

    if 'tipos_telefono' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'TIPOS_TELEFONO'
            FROM configs c
            WHERE c.stabla = 'TIPO_TELEFONO'
              AND c.nestado = 1;
    end if;

    if 'codigos_telefono_pais' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                p.nid_pais::text,
                CONCAT(p.sdescripcion, ' (+', p.scodigo_llamada_inter, ')'),
                'CODIGOS_TELEFONO_PAIS'
            FROM paises p
            WHERE p.scodigo_llamada_inter is not null
              AND p.nestado = 1;
    end if;

    if 'tipos_email' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'TIPOS_EMAIL'
            FROM configs c
            WHERE c.stabla = 'TIPO_EMAIL'
              AND c.nestado = 1;
    end if;

    if 'ubigeos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                u.nid_ubigeo::text,
                CONCAT(u.subigeo_departamento, '-', u.subigeo_provincia, '-', u.subigeo_distrito),
                'UBIGEOS'
            FROM ubigeos u
            WHERE u.nestado = 1;
    end if;

    if 'parentescos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'PARENTESCOS'
            FROM configs c
            WHERE c.stabla = 'PARENTESCO'
              AND c.nestado = 1;
    end if;

    if 'entidades_bancaria' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'ENTIDADES_BANCARIA'
            FROM configs c
            WHERE c.stabla = 'BANCO'
              AND c.nestado = 1;
    end if;

    if 'tipos_cuenta_bancaria' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'TIPOS_CUENTA_BANCARIA'
            FROM configs c
            WHERE c.stabla = 'TIPO_CUENTA'
              AND c.nestado = 1;
    end if;

    if 'sedes' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                nid_sede::text,
                sdescripcion,
                'SEDES'
            FROM sedes
            WHERE sentity = 'EMPRESA'
              AND nestado = 1;
    end if;

    if 'sedes_suggest' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                nid_sede::text,
                sdescripcion,
                'SEDES_SUGGEST'
            FROM sedes
            WHERE sentity = 'EMPRESA'
              AND nestado = 1;
    end if;

    if 'colaboradores_sin_contrato' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_colaborador::text,
            p.spersona_nombre,
            'COLABORADORES_SIN_CONTRATO'
        FROM colaboradores c
        JOIN  horarios_colaboradores hc on c.nid_colaborador = hc.nid_colaborador and hc.nestado = 1
        JOIN personas p on c.nid_persona = p.nid_persona
        JOIN usuarios u on p.nid_persona = u.nid_persona
        JOIN roles_usuarios ru on u.nid_usuario = ru.nid_usuario
        LEFT JOIN contratos c2 on c.nid_colaborador = c2.nid_colaborador
        WHERE c.nestado_colaborador = 1
            AND c2.nid_contrato is null
            AND ru.nid_rol not in (1, 10);
    end if;

    if 'colab_contrato_suggest' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_colaborador::text,
            p.spersona_nombre,
            'COLABORADORES_CON_CONTRATO'
        FROM colaboradores c
        JOIN  horarios_colaboradores hc on c.nid_colaborador = hc.nid_colaborador and hc.nestado = 1
        JOIN personas p on c.nid_persona = p.nid_persona
        JOIN usuarios u on p.nid_persona = u.nid_persona
        JOIN roles_usuarios ru on u.nid_usuario = ru.nid_usuario
        LEFT JOIN contratos c2 on c.nid_colaborador = c2.nid_colaborador
        WHERE c.nestado_colaborador = 1
            AND c2.nid_contrato is not null
            AND ru.nid_rol not in (1, 10);
    end if;

    if 'colaboradores_con_usuario' = any(var_configuration_request) then
        RETURN QUERY
            SELECT DISTINCT
                c.nid_colaborador::text,
                p.spersona_nombre,
                'COLABORADORES_CON_USUARIO'
            FROM colaboradores c
            JOIN  horarios_colaboradores hc on c.nid_colaborador = hc.nid_colaborador and hc.nestado = 1
            JOIN personas p on c.nid_persona = p.nid_persona
            JOIN usuarios u on p.nid_persona = u.nid_persona
            JOIN colaboradores_supervisor cs on c.nid_colaborador = cs.nid_colaborador
            WHERE c.nestado_colaborador = 1
                AND hc.nestado = 1
                AND cs.bprincipal = 1;
    end if;

    if 'colaboradores' = any(var_configuration_request) then
        if p_slug_listado = 'TODOS' OR p_slug_listado = '' then
            RETURN QUERY
            SELECT
                c.nid_colaborador::text,
                p.spersona_nombre,
                'COLABORADORES'
            FROM colaboradores c
            JOIN personas p on c.nid_persona = p.nid_persona
            JOIN usuarios u on p.nid_persona = u.nid_persona
            JOIN horarios_colaboradores hc on c.nid_colaborador = hc.nid_colaborador
            WHERE c.nestado_colaborador = 1
                AND hc.nestado = 1;
        end if;

        if p_slug_listado = 'EQUIPO' then
            CREATE TEMP TABLE temp_id_team_user_leader (nid_team integer);

            INSERT INTO temp_id_team_user_leader(nid_team)
            SELECT
                t.nid_team
            FROM team t
            WHERE t.nid_lider = p_collaborator_id
                OR t.nid_director = p_collaborator_id;

            RETURN QUERY
            SELECT DISTINCT
                tc.nid_colaborador::text,
                pc.spersona_nombre,
                'COLABORADORES'
            FROM team_colaboradors tc
            JOIN colaboradores c on tc.nid_colaborador = c.nid_colaborador
            JOIN personas_colaboradores pc on c.nid_colaborador = pc.nid_colaborador
            WHERE tc.nid_team IN (SELECT t.nid_team FROM temp_id_team_user_leader t)
                AND c.nestado_colaborador = 1;
        end if;

        if p_slug_listado = 'PROYECTO' then
            CREATE TEMP TABLE temp_id_projects (nid_project integer);

            INSERT INTO temp_id_projects(nid_project)
            SELECT DISTINCT
                pl.nid_proyecto
            FROM proyecto_lider pl
            WHERE pl.nid_lider = p_collaborator_id
                AND pl.nestado = 1

            UNION

            SELECT
                p.nid_proyecto
            FROM proyectos p
            WHERE p.nid_director = p_collaborator_id;

            RETURN QUERY
            SELECT DISTINCT
                c.nid_colaborador::text,
                pc.spersona_nombre,
                'COLABORADORES'
            FROM colaboradores c
            JOIN horarios_colaboradores hc on c.nid_colaborador = hc.nid_colaborador
            JOIN colaboradores_proyectos cp on c.nid_colaborador = cp.nid_colaborador
            JOIN personas_colaboradores pc on c.nid_colaborador = pc.nid_colaborador
            WHERE cp.nid_proyecto IN (SELECT nid_proyecto FROM temp_id_projects)
                AND hc.nestado = 1
                AND c.nestado_colaborador = 1
                AND c.nid_colaborador <> p_collaborator_id;

        end if;
    end if;

    if 'collaborator_self_update_information' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_colaborador::text,
            NULL,
            'COLLABORATOR-SELF-UPDATE'
        FROM colaboradores c
        JOIN usuarios u on c.nid_persona = u.nid_persona
        WHERE c.bselfupdate = 1 AND u.nid_usuario = p_user_id;
    end if;

    if 'colaboradores_suggest' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            scodigo,
            sdescription,
            stipo_configuration
        FROM collaborator_suggests_by_slug(p_slug_listado, p_collaborator_id);
    end if;

    if 'colaboradores_de_supervisor' = any(var_configuration_request) then
        RETURN QUERY
        SELECT DISTINCT
            c.nid_colaborador::text,
            p.spersona_nombre,
            'COLABORADORES_DE_SUPERVISOR'
        FROM colaboradores c
        JOIN personas p on c.nid_persona = p.nid_persona
        JOIN horarios_colaboradores hc on c.nid_colaborador = hc.nid_colaborador
        JOIN colaboradores_proyectos cp on c.nid_colaborador = cp.nid_colaborador
        JOIN proyectos p2 on cp.nid_proyecto = p2.nid_proyecto
        JOIN supervisor_proyecto sp on p2.nid_proyecto = sp.nid_proyecto
        JOIN coordinadores co on sp.nid_supervisor = co.nid_coordinador
        JOIN personas p3 on co.nid_persona = p3.nid_persona
        JOIN usuarios u on p3.nid_persona = u.nid_persona
        WHERE c.nestado_colaborador = 1
            AND hc.nestado = 1
            AND u.nid_usuario = p_user_id;
    end if;

    if 'clientes' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.nid_cliente::text,
            p.spersona_nombre,
            'CLIENTES'
        FROM clientes c
        LEFT JOIN personas p on c.nid_persona = p.nid_persona
        WHERE c.nestado = 1;
    end if;

    if 'horarios' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                h.nid_horario::text,
                h.sdescripcion,
                'HORARIO'
            FROM horarios h
            WHERE h.nestado = 1;
    end if;

    if 'horarios_suggest' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            h.nid_horario::text,
            h.sdescripcion,
            'HORARIO'
        FROM horarios h;
    end if;

    if 'dias_semana' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            c.scodigo,
            c.sdescripcion,
            'DIA_SEMANA'
        FROM configs c
        WHERE stabla = 'DIA_SEMANA'
            and nestado = 1;
    end if;

    if 'tipo_marcacion' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'TIPO_MARCACION'
            FROM configs c
            WHERE stabla = 'TIPO_MARCACION'
              and nestado = 1;
    end if;

    if 'tipos_rol_interno_externo' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            r.nid_rol::text,
            r.sdescripcion,
            CASE
                WHEN r.stipo_rol = '1' THEN concat('ROL_', c.sdescripcion)
                WHEN r.stipo_rol = '2' THEN concat('ROL_', c.sDescripcion)
            END
        FROM roles r
        JOIN configs c on r.stipo_rol = c.scodigo AND c.stabla = 'TIPO_ROL'
          WHERE r.nestado = 1;
    end if;

    if 'roles_suggest' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                r.nid_rol::text,
                r.sdescripcion,
                'TODO_ROLES'
            FROM roles r
            JOIN configs c on r.stipo_rol = c.scodigo AND c.stabla = 'TIPO_ROL'
            WHERE r.nestado = 1;
    end if;

    if 'personas_sin_usuario' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            p.nid_persona::text,
            p.spersona_nombre,
            CASE
                WHEN (
                    SELECT count(*)
                    FROM personas p2
                    JOIN colaboradores c on c.nid_persona = p2.nid_persona
                    WHERE p2.nid_persona = p.nid_persona
                    ) > 0 then 'SIN_USUARIO_COLABORADOR'

                WHEN (
                    SELECT count(*)
                    FROM personas p2
                    JOIN coordinadores c2 on p2.nid_persona = c2.nid_persona
                    WHERE p2.nId_Persona = p.nId_Persona) THEN 'SIN_USUARIO_COORDINADOR'
            END
        FROM personas p
        LEFT JOIN usuarios u on p.nid_persona = u.nid_persona
        WHERE
            u.nid_usuario is null
            AND p.nid_tipo_persona = 1
            AND ((
                SELECT
                    count(*)
                FROM personas p2
                JOIN colaboradores c on c.nid_persona = p2.nid_persona
                WHERE p2.nid_persona = p.nid_persona
            ) > 0
            OR (
                SELECT
                    count(*)
                FROM personas p2
                JOIN coordinadores c2 on p2.nid_persona = c2.nid_persona
                WHERE p2.nid_persona = p.nid_persona
            ) > 0);
    end if;

    if 'personas_con_usuario' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                p.nid_persona::text,
                p.spersona_nombre,
                'PERSONAS_CON_USUARIO'
            FROM personas p
            JOIN usuarios u on p.nid_persona = u.nid_persona
            WHERE p.nestado = 1;
    end if;

    if 'tipo_rol' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'TIPO_ROL'
            FROM configs c
            WHERE stabla = 'TIPO_ROL'
              and nestado = 1;
    end if;

    if 'categoria_tarea' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.nid_categoria::text,
                c.snombre,
                'CATEGORIA_TAREA'
            FROM categorias c
            WHERE c.nestado = 1;
    end if;

    if 'tipo_actividad' = any(var_configuration_request) then
        IF EXISTS (
            SELECT 1
            FROM colaboradores_proyectos cp
            WHERE cp.nid_proyecto = (select nid_proyecto from proyectos where snombre = 'ASEGURAMIENTO DE LA CALIDAD')
            AND cp.nid_colaborador = (
                SELECT
                    nid_colaborador
                FROM colaboradores c
                JOIN personas p on c.nid_persona = p.nid_persona
                JOIN usuarios u on p.nid_persona = u.nid_persona
                WHERE u.nid_usuario = p_user_id)
            ) then

            RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'TIPO_ACTIVIDAD'
            FROM configs c
            WHERE c.stabla = 'TIPO_ACTIVIDAD'
                AND c.nestado = 1
                AND c.sdescripcion like '%CALIDAD%';
        else
            RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'TIPO_ACTIVIDAD'
            FROM configs c
            WHERE c.stabla = 'TIPO_ACTIVIDAD'
              AND c.nestado = 1
              AND c.sdescripcion not like '%CALIDAD%';
        end if;

    end if;

    if 'proyectos_supervisor' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                pry.nid_proyecto::text,
                pry.sdescripcion,
                'PROYECTOS_SUPERVISOR'
            FROM (
                    SELECT
                        p.nid_proyecto::text,
                        CONCAT(p.snombre, ' ', '(',
                               p2.sprimer_nombre,  ')') as sdescripcion
                    FROM proyectos p
                    JOIN coordinadores_proyectos cp on p.nid_proyecto = cp.nid_proyecto
                    JOIN coordinadores c on cp.nid_coordinador = c.nid_coordinador
                    JOIN personas p2 on c.nid_persona = p2.nid_persona
                    JOIN supervisor_proyecto sp on cp.nid_proyecto = sp.nid_proyecto
                    JOIN coordinadores supervisor on sp.nid_supervisor = supervisor.nid_coordinador
                    JOIN personas p_sup on p_sup.nid_persona = supervisor.nid_persona
                    JOIN usuarios u_sup on p_sup.nid_persona = u_sup.nid_persona
                    WHERE u_sup.nid_usuario = p_user_id
                 ) AS pry
        WHERE pry.sdescripcion NOT LIKE '%SIN PROYECTO%';
    end if;

    if 'proyectos_coor' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                p.nid_proyecto::text,
                CONCAT(p.sNombre, ' ', '(',
                       p2.sPrimer_Nombre ,
                       ')'),
                'PROYECTOS_COOR'
            FROM proyectos p
            JOIN coordinadores_proyectos cp on p.nid_proyecto = cp.nid_proyecto
            JOIN coordinadores c on cp.nid_coordinador = c.nid_coordinador
            JOIN personas p2 on c.nid_persona = p2.nid_persona
            JOIN usuarios u on p2.nid_persona = u.nid_persona
            WHERE u.nid_usuario = p_user_id
            AND c.nestado = 1;
    end if;

    if 'proyectos_todos_masivo' = any(var_configuration_request) then
        RETURN QUERY
            select
                scodigo::text,
                sdescripcion,
                stipo_configuracion
            from ((select p.nid_proyecto  as scodigo,
                          CONCAT(p.snombre, ' ', '(', p2.sprimer_nombre , ')') as sdescripcion,
                          'PROYECTOS_TODOS_MASIVO' as stipo_configuracion
                   from proyectos p
                            join coordinadores_proyectos cp on p.nid_proyecto = cp.nid_proyecto
                            join coordinadores c on cp.nid_coordinador = c.nid_coordinador
                            join personas p2 on c.nid_persona = p2.nid_persona
                  )
                  union
                  (SELECT p.nid_proyecto         AS scodigo,
                          CONCAT(
                                  p.scodigo ,' ',p.snombre, ' ','(',p2.sprimer_nombre ,')'
                          )               AS sdescripcion,
                          'PROYECTOS_CON_ACCESO' AS stipo_configuracion
                   FROM proyectos p
                            JOIN coordinadores_proyectos cp ON p.nid_proyecto = cp.nid_proyecto
                            JOIN coordinadores c ON cp.nid_coordinador = c.nid_coordinador
                            JOIN personas p2 ON c.nid_persona = p2.nid_persona
                   WHERE p.nestado = 1
                     AND (
                       p.ntipo_acceso = 0
                           OR (
                           p.ntipo_acceso = 1
                               AND EXISTS (SELECT 1
                                           FROM colaboradores_proyectos cop
                                           WHERE cop.nid_proyecto = p.nid_proyecto
                                             AND cop.nid_proyecto = p_collaborator_id)
                               OR EXISTS (SELECT 1
                                          FROM proyecto_lider pl
                                          WHERE pl.nid_proyecto = p.nid_proyecto
                                            AND pl.nid_lider = p_collaborator_id)
                           )
                       ))

                  UNION

                  (select p.nid_proyecto         as scodigo,
                          CONCAT(p.snombre, ' ', '(', p3.sprimer_nombre , ')') as sdescripcion,
                          'PROYECTOS_COLABORADOR'                               as stipo_configuracion
                   from proyectos p
                            join coordinadores_proyectos cp on p.nid_proyecto = cp.nid_proyecto
                            join coordinadores c on cp.nid_coordinador = c.nid_coordinador
                            join personas p2 on c.nid_persona = p2.nid_persona
                            join colaboradores_proyectos cp2 on p.nid_proyecto = cp2.nid_proyecto
                            join colaboradores c2 on cp2.nid_colaborador = c2.nid_colaborador
                            join personas p3 on c2.nid_persona = p3.nid_persona
                            join usuarios u on p3.nid_persona = u.nid_persona
                   where u.nid_usuario = p_user_id
                     and p.nestado = 1)
                  union
                  (select p.nid_proyecto                                        as scodigo,
                          CONCAT(p.snombre, ' ', '(', p2.sprimer_nombre , ')') as sdescripcion,
                          'PROYECTOS_LIDER_DIRECTOR'                            as stipo_Configuracion
                   from proyectos p
                            join proyecto_lider pl on p.nid_proyecto = pl.nid_proyecto and pl.nestado = 1--PRY_LIDER
                            join coordinadores_proyectos cp on p.nid_proyecto = cp.nid_proyecto
                            join coordinadores c on cp.nid_coordinador = c.nid_coordinador
                            join personas p2 on c.nid_persona = p2.nid_persona where (p.nid_proyecto = p_collaborator_id
                      or pl.nid_lider = p_collaborator_id)
                                                                                 and p.nestado = 1)
                  union
                  (select p.nid_proyecto                                        as scodigo,
                          CONCAT(p.snombre, ' ', '(', p_coor.sprimer_nombre , ')') as sdescripcion,
                          'PROYECTOS_COORDINADOR'                               as stipo_configuracion
                   from proyectos p
                            join coordinadores_proyectos cp on cp.nid_proyecto = p.nid_proyecto
                            join coordinadores coo on coo.nid_coordinador = cp.nid_coordinador
                            join personas p_coor on p_coor.nid_persona = coo.nid_persona
                            join usuarios u on u.nid_persona = p_coor.nid_persona
                   where u.nid_usuario = p_user_id
                     and p.nestado = 1)) as proyectos
            where sdescripcion not like '%SIN PROYECTO%'::text;
    end if;



    if 'proyectos' = any(var_configuration_request) then
        RETURN QUERY
            select
                scodigo::text,
                sdescripcion,
                stipo_configuracion
            from ((select p.nid_proyecto                                        as scodigo,
                          CONCAT(p.snombre, ' ', '(', p2.sprimer_nombre, ')') as sdescripcion,
                          'PROYECTOS'                   as stipo_configuracion
                   from proyectos p
                            join coordinadores_proyectos cp on p.nid_proyecto = cp.nid_proyecto
                            join coordinadores c on cp.nid_coordinador = c.nid_coordinador
                            join personas p2 on c.nid_persona = p2.nid_persona

                   where p.nestado = 1)
                  union
                  (SELECT p.nid_proyecto         AS scodigo,

                          CONCAT(
                                  p.snombre, ' ',
                                  '(',
                                  p2.sprimer_nombre,
                                  ')'
                          )                      AS sdescripcion,
                          'PROYECTOS_CON_ACCESO' AS stipo_configuracion
                   FROM proyectos p
                            JOIN coordinadores_proyectos cp ON p.nid_proyecto = cp.nid_proyecto
                            JOIN Coordinadores c ON cp.nid_coordinador = c.nid_coordinador
                            JOIN personas p2 ON c.nid_persona = p2.nid_persona
                            LEFT JOIN configs c1 ON c1.stabla = 'PREFIJO_PROYECTO'
                       AND c1.scodigo = CASE
                                            WHEN p.sprefijo = 1 THEN '1'
                                            WHEN p.sprefijo = 2 THEN '2'
                                            ELSE NULL
                           END

                   WHERE p.nestado = 1
                     AND (p.ntipo_acceso = 0
                       OR (
                              p.ntipo_acceso = 1
                                  AND EXISTS (SELECT 1
                                              FROM colaboradores_proyectos cop
                                              WHERE cop.nid_proyecto = p.nid_proyecto
                                                AND cop.nid_colaborador = p_collaborator_id)
                                  OR EXISTS (SELECT 1
                                             FROM proyecto_lider pl
                                             WHERE pl.nid_proyecto = p.nid_proyecto
                                               AND pl.nid_lider = p_collaborator_id)
                              )
                       ))

                  UNION

                  (select p.nid_proyecto                                        as scodigo,
                          CONCAT(p.snombre, ' ', '(', p2.sprimer_nombre, ')') as sdescripcion,
                          'PROYECTOS_COLABORADOR'                               as stipo_configuracion
                   from proyectos p
                            join coordinadores_proyectos cp on p.nid_proyecto = cp.nid_proyecto
                            join coordinadores c on cp.nid_coordinador = c.nid_coordinador
                            join personas p2 on c.nid_persona = p2.nid_persona
                            join colaboradores_proyectos cp2 on p.nid_proyecto = cp2.nid_proyecto
                            join colaboradores c2 on cp2.nid_colaborador = c2.nid_colaborador
                            join personas p3 on c2.nid_persona = p3.nid_persona
                            join usuarios u on p3.nid_persona = u.nid_persona
                   where u.nid_usuario = p_user_id
                     and p.nestado = 1)
                  union
                  (select p.nid_proyecto           as scodigo,
                          CONCAT(p.snombre, ' ', '(', p2.sprimer_nombre , ')') as sdescripcion,
                          'PROYECTOS_LIDER_DIRECTOR'                  as stipo_configuracion
                   from proyectos p
                            join proyecto_lider pl on p.nid_proyecto = pl.nid_proyecto and pl.nestado = 1--PRY_LIDER
                            join coordinadores_proyectos cp on p.nid_proyecto = cp.nid_proyecto
                            join coordinadores c on cp.nid_coordinador = c.nid_coordinador
                            join personas p2 on c.nid_persona = p2.nid_persona
                   where (p.nid_lider = p_collaborator_id
                       or pl.nid_lider = p_collaborator_id)
                     and p.nestado = 1)
                  union
                  (select p.nid_proyecto                                        as scodigo,
                          CONCAT(p.snombre, ' ', '(', p_coor.sprimer_nombre, ')') as sdescripcion,
                          'PROYECTOS_COORDINADOR' as stipo_configuracion
                   from proyectos p
                            join coordinadores_proyectos cp on cp.nid_proyecto = p.nid_proyecto
                            join coordinadores coo on coo.nid_coordinador = cp.nid_coordinador
                            join personas p_coor on p_coor.nid_persona = coo.nid_persona
                            join usuarios u on u.nid_persona = p_coor.nid_persona
                   where u.nid_usuario = p_user_id
                     and p.nestado = 1)) as proyectos
            where sdescripcion not like '%SIN PROYECTO%';
    end if;

    if 'proyectos_colaborador_v2' = any(var_configuration_request) then
        RETURN QUERY
            SELECT p.nid_proyecto::text,
                   CONCAT(p.snombre, ' ', '(', pc.sprimer_nombre , ')'),
                   'PROYECTOS_COLABORADOR_V2'
            FROM proyectos p
            JOIN coordinadores_proyectos cp ON cp.nid_proyecto = p.nid_proyecto
            JOIN coordinadores coor ON coor.nid_coordinador = cp.nid_coordinador
            JOIN personas pc ON pc.nid_persona = coor.nid_persona
            WHERE p.nid_proyecto IN (SELECT t.nid_proyecto
                                     FROM tareas t
                                     WHERE t.nid_colaborador = p_collaborator_id);
    end if;

    if 'requerimientos_colaborador' = any(var_configuration_request) then
        SELECT r.nid_requerimiento::text,
               r.snombre,
               'REQUERIMIENTOS_COLABORADOR'
        FROM requerimientos r
        WHERE r.nid_proyecto IN (SELECT t.nid_proyecto
                                 FROM tareas t
                                 WHERE t.nid_colaborador = p_collaborator_id);
    end if;

    if 'proyectos_todos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT p.nid_proyecto::text,
                   p.snombre,
                   'PROYECTOS_TODOS'
            FROM proyectos p
            WHERE sdescripcion NOT LIKE '%SIN PROYECTO%';
    end if;

    if 'pry_req_colaborador' = any(var_configuration_request) then
        RETURN QUERY
            SELECT '0',
                   CONCAT(r.nid_proyecto, '-', r.nid_requerimiento),
                   'PRY_REQ_COLABORADOR'
            FROM requerimientos r
            WHERE r.nid_proyecto IN (SELECT t.nid_proyecto
                                     FROM tareas t
                                     WHERE t.nid_colaborador = p_collaborator_id);
    end if;

    if 'requerimientos_por_proyecto' = any(var_configuration_request) then
        RETURN QUERY
            SELECT r.nid_requerimiento::text,
                   r.snombre,
                   'REQUERIMIENTOS_POR_PROYECTO'
            FROM proyectos p
            JOIN requerimientos r ON r.nid_proyecto = p.nid_proyecto
            WHERE p.nid_proyecto IN (SELECT p.nid_proyecto
                                     FROM proyectos p
                                     JOIN proyecto_lider pl ON pl.nid_proyecto = p.nid_proyecto
                                     WHERE p.nid_lider = p_collaborator_id
                                       and pl.nid_lider = p_collaborator_id
                                       and pl.nestado = 1);
    end if;

    if 'pry_req_por_proyecto' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                '0',
                CONCAT(p.nid_proyecto, '-', r.nid_requerimiento),
                'PRY_REQ_POR_PROYECTO'
            FROM proyectos p
            JOIN requerimientos r ON r.nid_proyecto = p.nid_proyecto
            WHERE p.nid_proyecto IN (SELECT p.nid_proyecto
                                     FROM proyectos p
                                     JOIN proyecto_lider pl ON pl.nid_proyecto = p.nid_proyecto
                                     WHERE p.nid_lider = p_collaborator_id
                                       and pl.nid_lider = p_collaborator_id
                                       and pl.nestado = 1);
    end if;

    if 'proyectos_team' = any(var_configuration_request) then
        RETURN QUERY
            SELECT p.nid_proyecto::text,
                   CONCAT(p.snombre, ' ', '(', pc.sprimer_nombre, ')'),
                   'PROYECTOS_TEAM'
            FROM proyectos p
            JOIN coordinadores_proyectos cp ON cp.nid_proyecto = p.nid_proyecto
            JOIN coordinadores coor ON coor.nid_coordinador = cp.nid_coordinador
            JOIN personas pc ON pc.nid_persona = coor.nid_persona
            WHERE p.nid_proyecto IN (SELECT tar.nid_proyecto
                                     FROM team t
                                     JOIN team_colaboradors tc ON t.nid_team = tc.nid_team
                                     JOIN tareas tar ON tar.nid_colaborador = tc.nid_colaborador
                                     WHERE t.nid_lider = p_collaborator_id
                                        OR t.nid_director = p_collaborator_id);
    end if;

    if 'requerimientos_team' = any(var_configuration_request) then
        RETURN QUERY
            SELECT r.nid_requerimiento::text,
                   r.snombre,
                   'REQUERIMIENTOS_TEAM'
            FROM proyectos p
            JOIN requerimientos r ON r.nid_proyecto = p.nid_proyecto
            WHERE r.nid_proyecto IN (SELECT tar.nid_proyecto
                                     FROM team t
                                     JOIN team_colaboradors tc ON t.nid_team = tc.nid_team
                                     JOIN tareas tar ON tar.nid_colaborador = tc.nid_colaborador
                                     WHERE t.nid_lider = p_collaborator_id
                                        OR t.nid_director = p_collaborator_id);
    end if;

    if 'proyectos_requerimientos_team' = any(var_configuration_request) then
        RETURN QUERY
            SELECT '0',
                   CONCAT(p.nid_proyecto, '-', r.nid_requerimiento),
                   'PROYECTOS_REQUERIMIENTOS_TEAM'
            FROM proyectos p
            JOIN requerimientos r ON r.nid_proyecto = p.nid_proyecto
            WHERE r.nid_proyecto IN (SELECT tar.nid_proyecto
                                     FROM team t
                                     JOIN team_colaboradors tc ON t.nid_team = tc.nid_team
                                     JOIN tareas tar ON tar.nid_colaborador = tc.nid_colaborador
                                     WHERE t.nid_lider = p_collaborator_id
                                        OR t.nid_director = p_collaborator_id);
    end if;

    if 'proyectos_requerimientos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                '0',
                CONCAT(p.nid_proyecto, '-', r.nid_requerimiento),
                'PROYECTOS_REQUERIMIENTOS'
            FROM proyectos p
            JOIN requerimientos r ON r.nid_proyecto = p.nid_proyecto;
    end if;

    if 'proyectos_en_equipo_proyecto' = any(var_configuration_request) then
        RETURN QUERY
            SELECT DISTINCT p.nid_proyecto::text,
                            CONCAT(p.snombre, ' ', '(', p2.sprimer_nombre, ')'),
                            'PROYECTOS_EN_EQUIPO_PROYECTO'
            FROM proyectos p
            JOIN proyecto_lider pl ON p.nid_proyecto = pl.nid_proyecto AND pl.nestado = 1 --PRY_LIDER
            -- coordinator name
            JOIN coordinadores_proyectos cp ON p.nid_proyecto = cp.nid_proyecto
            JOIN coordinadores c ON cp.nid_coordinador = c.nid_coordinador
            JOIN personas p2 ON c.nid_persona = p2.nid_persona
            WHERE ((p.nid_lider = p_collaborator_id OR pl.nid_lider = p_collaborator_id) AND
                   p.nestado = 1) -- leader's project
               OR p.nid_proyecto IN ( -- leader's task project
                SELECT tar.nid_proyecto
                FROM team t
                JOIN team_colaboradors tc ON t.nid_team = tc.nid_team
                JOIN tareas tar ON tar.nid_colaborador = tc.nid_colaborador
                WHERE t.nid_lider = p_collaborator_id
                   OR t.nid_director = p_collaborator_id);
    end if;

    if 'requerimientos_en_equipo_proyecto' = any(var_configuration_request) then
        RETURN QUERY
            SELECT r.nid_requerimiento::text,
                   r.snombre,
                   'REQUERIMIENTOS_EN_EQUIPO_PROYECTO'
            FROM proyectos p
            JOIN requerimientos r ON r.nid_proyecto = p.nid_proyecto
            WHERE r.nid_proyecto IN (SELECT tar.nid_proyecto
                                     FROM team t
                                     JOIN team_colaboradors tc ON t.nid_team = tc.nid_team
                                     JOIN tareas tar ON tar.nid_colaborador = tc.nid_colaborador
                                     WHERE t.nid_lider = p_collaborator_id
                                        OR t.nid_director = p_collaborator_id)
               OR p.nid_proyecto IN (SELECT p.nid_proyecto
                                     FROM proyectos p
                                     JOIN proyecto_lider pl ON pl.nid_proyecto = p.nid_proyecto and pl.nestado = 1
                                     WHERE p.nid_lider = p_collaborator_id);
    end if;

    if 'pry_req_en_equipo_proyecto' = any(var_configuration_request) then
        RETURN QUERY
            SELECT '0',
                   CONCAT(p.nid_proyecto, '-', r.nid_requerimiento),
                   'PRY_REQ_EN_EQUIPO_PROYECTO'
            FROM proyectos p
            JOIN requerimientos r ON r.nid_proyecto = p.nid_proyecto
            WHERE r.nid_proyecto IN (SELECT tar.nid_proyecto
                                     FROM team t
                                     JOIN team_colaboradors tc ON t.nid_team = tc.nid_team
                                     JOIN tareas tar ON tar.nid_colaborador = tc.nid_colaborador
                                     WHERE t.nid_lider = p_collaborator_id
                                        OR t.nid_director = p_collaborator_id)
               OR p.nid_proyecto IN (SELECT p.nid_proyecto
                                     FROM proyectos p
                                     JOIN proyecto_lider pl ON pl.nid_proyecto = p.nid_proyecto and pl.nestado = 1
                                     WHERE p.nid_lider = p_collaborator_id);
    end if;

    if 'proyectos_suggest' = any(var_configuration_request) then
        if p_slug_listado = 'TODOS' then
            RETURN QUERY
                SELECT p.nid_proyecto::text,
                       p.snombre,
                       'PROYECTOS_SUGERENCIAS'
                FROM proyectos p
                GROUP BY p.snombre, p.scodigo, p.nid_proyecto;
        end if;

        if p_slug_listado = 'PROYECTO' then
            RETURN QUERY
                SELECT '1',
                       p.snombre,
                       'PROYECTOS_SUGERENCIAS'
                FROM proyectos p
                JOIN requerimientos r on p.nid_proyecto = r.nid_proyecto
                JOIN proyecto_lider pl on pl.nid_proyecto = p.nid_proyecto
                WHERE (p.nid_director = p_collaborator_id or pl.nid_lider = p_collaborator_id)
                  and pl.nestado = 1
                GROUP by p.nid_proyecto, p.snombre , p.scodigo;
        end if;

        if p_slug_listado = 'EQUIPO' then
            RETURN QUERY
                SELECT '1',
                       p.snombre,
                       'PROYECTOS_SUGERENCIAS'
                FROM proyectos p
                JOIN colaboradores_proyectos cp ON cp.nid_proyecto = p.nid_proyecto
                JOIN team_colaboradors tc ON tc.nid_colaborador = cp.nid_colaborador
                JOIN team t ON t.nid_team = tc.nId_team
                JOIN personas_colaboradores pc ON pc.nid_colaborador = t.nid_lider
                JOIN usuarios u ON u.nid_persona = pc.nid_persona
                WHERE u.nid_usuario = p_user_id
                GROUP BY p.snombre, p.scodigo;
        end if;

    end if;

    if 'requerimientos_suggest' = any(var_configuration_request) then
        if p_slug_listado = 'TODOS' then
            RETURN QUERY
                SELECT '1',
                       r.snombre,
                       'REQUERIMIENTOS_SUGERENCIAS'
                FROM requerimientos r
                GROUP BY r.snombre , r.scodigo;
        end if;

        if p_slug_listado = 'PROYECTO' then
            RETURN QUERY
                SELECT '1',
                       r.snombre,
                       'REQUERIMIENTOS_SUGERENCIAS'
                FROM requerimientos r
                WHERE nid_lider = p_collaborator_id
                GROUP by snombre , scodigo;
        end if;

        if p_slug_listado = 'EQUIPO' then
            RETURN QUERY
                SELECT '1',
                       r.snombre,
                       'REQUERIMIENTOS_SUGERENCIAS'
                FROM requerimientos r
                JOIN proyectos p ON p.nid_proyecto = r.nid_proyecto
                JOIN colaboradores_proyectos cp ON cp.nid_proyecto = p.nid_proyecto
                JOIN team_colaboradors tc ON tc.nid_colaborador = cp.nid_colaborador
                JOIN team t ON t.nid_team = tc.nid_team
                JOIN personas_colaboradores pc ON pc.nid_colaborador = t.nid_lider
                JOIN usuarios u ON u.nid_persona = pc.nid_persona
                WHERE u.nid_usuario = p_user_id
                GROUP BY r.snombre, r.scodigo;
        end if;

    end if;

    if 'tipo_hora' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.scodigo,
                   c.sdescripcion,
                   'TIPO_HORA'
            FROM configs c
            WHERE c.stabla = 'TIPO_HORA'
              AND c.nestado = 1;
    end if;

    if 'tipo_servicio' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.scodigo,
                   c.sdescripcion,
                   'TIPO_SERVICIO'
            FROM configs c
            WHERE stabla = 'TIPO_SERVICIO';
    end if;

    if 'cliente_servicio_proyecto' = any(var_configuration_request) then
        RETURN QUERY
            select '0',
                   CONCAT(c.nid_cliente, '-', p.sid_tipo_servicio, '-', p.nid_proyecto),
                   'CLIENTE_SERVICIO_PROYECTO'
            from proyectos p
            join coordinadores_proyectos cp on p.nid_proyecto = cp.nid_proyecto
            join coordinadores c on cp.nid_coordinador = c.nid_coordinador;
    end if;

    if 'tipo_motivo' = any(var_configuration_request) then
        RETURN QUERY
            SELECT nId_Tipo_Motivo::text,
                   CONCAT(tm.sdescripcion, '-', tm.nrecuperable),
                   'TIPO_MOTIVO'
            from tipos_motivos as tm
            where nestado = 1;
    end if;

    if 'tipo_solicitud' = any(var_configuration_request) then
        RETURN QUERY
            SELECT nid_tipo_solicitud::text,
                   concat(sdescripcion, '-', nid_categoria),
                   'TIPO_SOLICITUD'
            from tipos_solicitudes
            where nestado = 1;
        --AND nId_Tipo_Solicitud NOT IN (26)
        -- AND nId_Tipo_Solicitud NOT IN (21, 20)
    end if;

    if 'feriados' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            nid_feriado::text,
            dfecha::text,
            'FERIADOS'
        FROM feriados
        WHERE nestado = 1;
    end if;

    if 'frontend_version' = any(var_configuration_request) then
        RETURN QUERY
        SELECT
            nid_v::text,
            sversion::text,
            'FRONTEND_VERSION'
        FROM version
        WHERE nestado = 1
        AND snombre_version = 'FRONTEND_VERSION'
        ORDER BY nid_v DESC
        LIMIT 1;
    end if;

    if 'usuarios' = any(var_configuration_request) then
        RETURN QUERY
            SELECT u.nid_usuario::text,
                   p.spersona_nombre,
                   'USUARIOS'
            from usuarios u
            join personas p on u.nid_persona = p.nid_persona
            where u.nestado = 1;
    end if;

    if 'usuarios_suggest' = any(var_configuration_request) then
        RETURN QUERY
            select u.nid_usuario::text,
                   p.spersona_nombre,
                   'USUARIOS'
            from usuarios u
            join personas p on u.nid_persona = p.nid_persona
            where u.nestado = 1
              and p.spersona_nombre <> 'Smart'
              and p.spersona_nombre NOT LIKE '%ADM - %';
    end if;

    if 'extemporanea' = any(var_configuration_request) then
        RETURN QUERY
            SELECT DISTINCT
                '1',
                dfecha_jornada::text,
                'EXTEMPORANEA'
            FROM marcas m
            JOIN colaboradores c on c.nid_colaborador = m.nid_colaborador
            JOIN personas p  on p.nid_persona =  c.nid_persona
            JOIN usuarios u on u.nid_persona = p.nid_persona
            WHERE m.nestado = 2
              and u.nid_usuario = p_user_id;
    end if;

    if 'requerimientos_activos_inactivos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT r.nid_requerimiento::text,
                   r.sNombre,
                   'REQUERIMIENTOS_ACTIVOS_INACTIVOS'
            FROM requerimientos r
            WHERE r.nid_proyecto = p_project_id;
    end if;

    if 'requerimientos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT r.nid_requerimiento::text,
                   r.snombre,
                   'REQUERIMIENTO'
            FROM requerimientos r
            WHERE r.nid_proyecto = p_project_id
              and r.nestado = 1;
    end if;

    if 'equipos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                t.nid_team,
                CONCAT(
                        p_lider.sprimer_nombre, ' ',
                        p_lider.sape_paterno, ', ',
                        p_director.sprimer_nombre, ' ',
                        p_director.sape_paterno),
                'EQUIPOS'
            FROM team t
            JOIN colaboradores c_lider on t.nid_lider = c_lider.nid_colaborador
            JOIN personas p_lider on c_lider.nid_persona = p_lider.nid_persona
            JOIN colaboradores c_director on t.nid_director = c_director.nid_colaborador
            JOIN personas p_director on c_director.nid_persona = p_director.nid_persona
            WHERE t.nestado = 1;
    end if;

    if 'supervisores_todos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                NULL AS scodigo,
                'AUTOGESTIONADO' AS sdescripcion,
                'SUPERVISORES_TODOS' AS stipo_configuracion

            UNION

            SELECT
                c.nId_Colaborador::text AS scodigo,
                p_lider.sPersona_Nombre AS sdescripcion,
                'SUPERVISORES_TODOS' AS stipo_configuracion
            FROM colaboradores c
            JOIN personas p_lider ON c.nid_persona = p_lider.nid_persona
            WHERE c.nestado_colaborador = 1 AND c.nestado = 1
              AND p_lider.spersona_nombre NOT LIKE '%ADM -%';
    end if;

    if 'supervisores_asignados' = any(var_configuration_request) then
        RETURN QUERY
            SELECT sa.nid_supervisor::text,
                   p_lider.spersona_nombre,
                   'SUPERVISORES_ASIGNADOS'
            FROM v_supervisores_asginados sa
            join colaboradores c_lider on sa.nid_supervisor  = c_lider.nId_Colaborador
            join personas p_lider on c_lider.nId_Persona = p_lider.nId_Persona;
    end if;

    if 'equipos_lider' = any(var_configuration_request) then
        RETURN QUERY
            SELECT t.nid_team::text,
                   CONCAT(
                           p_lider.sprimer_nombre, ' ',
                           p_lider.sape_paterno, ', ',
                           p_director.sprimer_nombre, ' ',
                           p_director.sape_paterno),
                   'EQUIPOS_LIDER'
            FROM team t
            JOIN colaboradores c_lider on t.nid_lider = c_lider.nid_colaborador
            JOIN personas p_lider on c_lider.nid_persona = p_lider.nid_persona
            JOIN colaboradores c_director on t.nid_director = c_director.nid_colaborador
            JOIN personas p_director on c_director.nid_persona = p_director.nid_persona
            WHERE t.nid_lider = p_collaborator_id
               OR t.nid_director = p_collaborator_id
                AND t.nestado = 1;
    end if;

    if 'equipos_supervisor' = any(var_configuration_request) then
        RETURN QUERY
            SELECT DISTINCT CONCAT(pry.nid_lider, '/', pry.nid_director),
                            CONCAT(p_lider.sprimer_nombre, ' ',
                                   p_lider.sape_paterno, ', ',
                                   p_director.sprimer_nombre,
                                   ' ', p_director.sape_paterno),
                            'EQUIPOS_SUPERVISOR'
            FROM proyectos pry
            JOIN colaboradores c_lider on pry.nid_lider = c_lider.nid_colaborador
            JOIN personas p_lider on c_lider.nid_persona = p_lider.nid_persona
            JOIN colaboradores c_director on pry.nid_director = c_director.nid_colaborador
            JOIN personas p_director on c_director.nid_persona = p_director.nid_persona

            JOIN coordinadores_proyectos coor_pry on coor_pry.nid_proyecto = pry.nid_proyecto
            JOIN supervisor_proyecto sup_pry on sup_pry.nid_proyecto = coor_pry.nid_proyecto

            JOIN coordinadores supervisor on supervisor.nid_coordinador = sup_pry.nid_supervisor
            JOIN personas p_sup on p_sup.nid_persona = supervisor.nid_persona
            JOIN usuarios u_sup on u_sup.nid_persona = p_sup.nid_persona
            WHERE u_sup.nid_usuario = p_user_id;
    end if;

    if 'fechas_cierre_asistencia' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                t.scodigo::text,
                t.sdescripcion,
                t.stipo_configuracion
            FROM (SELECT car.nid_cierre              as scodigo,
                         CONCAT(car.dfecha_inicio_asistencia, '/',
                                car.dfecha_fin_asistencia) as sdescripcion,
                         'FECHAS_CIERRE_ASISTENCIA'        as stipo_configuracion
                  FROM cierreasistenciareportes car
                  UNION
                  (SELECT car.nid_cierre           as scodigo,
                          CONCAT(car.dfecha_inicio_tardanza, '/',
                                 car.dfecha_fin_tardanza) as sdescripcion,
                          'FECHAS_CIERRE_TARDANZA'        as stipo_configuracion
                   FROM cierreasistenciareportes car)
                  UNION
                  SELECT car.nid_cierre             AS scodigo,
                         CONCAT(
                                 to_char(car.dfecha_inicio_tardanza, 'dd/MM/yyyy'), ' - ',
                                 to_char(car.dfecha_fin_tardanza, 'dd/MM/yyyy')
                         )                                AS sdescripcion,
                         'FECHAS_CIERRE_TARDANZA_SUGGESS' AS stipo_configuracion
                  FROM cierreasistenciareportes car
                  UNION
                  (SELECT car.nid_cierre   as scodigo,
                          car.dfecha_periodo::text as sdescripcion,
                          'FECHAS_CIERRE_PERIODO'  as stipo_configuracion
                   FROM cierreasistenciareportes car)) t
            ORDER BY t.scodigo DESC;
    end if;

    if 'metodologias' = any(var_configuration_request) then
        RETURN QUERY
            SELECT m.nid_metodologia::text as scodigo,
                   m.snombre               as sdescripcion,
                   'METODOLOGIAS_ACTIVAS'  as stipo_configuracion
            FROM metodologias m
            WHERE m.nestado = 1
            UNION
            (SELECT m.nid_metodologia::text  as scodigo,
                    m.snombre                as sdescripcion,
                    'METODOLOGIAS_INACTIVAS' as stipo_configuracion
             FROM metodologias m
             WHERE m.nestado = 0)
            UNION
            (SELECT c2.nid_categoria::text       as scodigo,
                    c2.snombre             as sdescripcion,
                    'CATEGORIAS_INACTIVAS' as stipo_configuracion
             FROM categorias c2
             WHERE c2.nestado = 0)
            UNION
            (SELECT c2.nid_categoria::text     as scodigo,
                    c2.snombre           as sdescripcion,
                    'CATEGORIAS_ACTIVAS' as stipo_configuracion
             FROM categorias c2
             WHERE c2.nestado = 1)
            UNION
            (SELECT CONCAT(m2.nid_metodologia, c.nid_categoria)      as scodigo,
                    CONCAT(m2.nid_metodologia, '-', c.nid_categoria) as sdescripcion,
                    'METODOLOGIAS_CATEGORIAS'                        as stipo_configuracion
             FROM metodologias_categorias mc
             JOIN categorias c on mc.nid_categoria = c.nid_categoria
             JOIN metodologias m2 on m2.nid_metodologia = mc.nid_metodologia);
    end if;

    if 'categorias' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.nid_categoria::text,
                   c.sdescripcion,
                   'CATEGORIAS'
            FROM requerimientos_categoria rc
            JOIN categorias c on c.nid_categoria = rc.nid_categoria
            WHERE rc.nid_requerimiento = p_requirement_id
              AND c.nestado = 1;
    end if;

    if 'categorias_suggest' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.nid_categoria::text,
                   c.sdescripcion,
                   'CATEGORIAS'
            FROM categorias c
            WHERE c.nestado = 1;
    end if;

    if 'clientes_suggest' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.nid_cliente::text,
                   p.spersona_nombre,
                   'CLIENTES_EMPRESA'
            FROM clientes c
            JOIN personas p ON p.nid_persona = c.nid_persona;
    end if;

    if 'categorias_solicitudes' = any(var_configuration_request) then
        RETURN QUERY
            SELECT cp.nid_categoria::text   AS scodigo,
                   cp.snombre               AS sdescripcion,
                   'CATEGORIAS_SOLICITUDES' AS stipo_configuracion
            FROM categorias_permisos cp
            WHERE cp.nestado = 1
            UNION
            (SELECT ts.nid_tipo_solicitud::text AS scodigo,
                    CONCAT(ts.sdescripcion,
                           '-', ts.nid_categoria,
                           '-', ts.ntipo_frecuencia,
                           '-', ts.nrangomin,
                           '-', ts.nrangomax,
                           '-', ts.nrangominadm,
                           '-', ts.nrangomaxadm,
                           '-', ts.narchivoobligatorio,
                           '-', ts.nafecta_asistencia,
                           '-', ts.nmostrar_calendario,
                           '-', ts.ntiempoanticipacion,
                           '-', ts.ndiascalendario,
                           '-', ts.nmostrar_selector) AS sdescripcion,
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS stipo_configuracion
             FROM tipos_solicitudes ts
             WHERE ts.nestado = 1
               --and ts.nId_Tipo_Solicitud NOT IN (26)
               and ts.nid_categoria NOT IN (2))
            UNION
            (select ts.nid_tipo_solicitud::text AS scodigo,
                    CONCAT(ts.sdescripcion,
                           '-', ts.nid_categoria,
                           '-', ts.ntipo_frecuencia,
                           '-', ts.nrangomin,
                           '-',
                           CASE
                               WHEN ts.nid_tipo_solicitud in (3,10) THEN ROUND(
                                       svc.ddias_vencidos_laborales, 0)
                               WHEN ts.nid_tipo_solicitud in (22) THEN (TRUNC(svc.ddias_generados_laborales, 0))
                           END,
                           '-', ts.nrangominadm,
                           '-',
                           CASE
                               WHEN ts.nid_tipo_solicitud in (3,10) THEN ROUND(
                                       svc.ddias_vencidos_laborales, 0)
                               WHEN ts.nid_tipo_solicitud in (22) THEN (TRUNC(svc.dDias_Generados_Laborales, 0))
                               END,
                           '-', ts.narchivoobligatorio,
                           '-', ts.nafecta_asistencia,
                           '-', ts.nmostrar_calendario,
                           '-', ts.ntiempoanticipacion,
                           '-', ts.ndiascalendario,
                           '-', ts.nmostrar_selector) AS sdescripcion,
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS stipo_configuracion
             from tipos_solicitudes ts
             join saldos_vacaciones_colaboradores_prueba svc
                           on svc.nid_colaborador = p_collaborator_id
             where ts.nestado = 1
               and ts.nid_categoria IN (2));
    end if;

    if 'categorias_solicitudess' = any(var_configuration_request) then
        RETURN QUERY
            SELECT cp.nid_categoria::text   AS scodigo,
                   cp.snombre               AS sdescripcion,
                   'CATEGORIAS_SOLICITUDES' AS stipo_configuracion
            FROM categorias_permisos cp
            WHERE cp.nestado = 1
            UNION
            (SELECT ts.nid_tipo_solicitud::text AS scodigo,
                    CONCAT(ts.sdescripcion,
                           '-', ts.nid_categoria,
                           '-', ts.ntipo_frecuencia,
                           '-', ts.nrangomin,
                           '-', ts.nrangomax,
                           '-', ts.nrangominadm,
                           '-', ts.nrangomaxadm,
                           '-', ts.narchivoobligatorio,
                           '-', ts.nafecta_asistencia,
                           '-', ts.nmostrar_calendario,
                           '-', ts.ntiempoanticipacion,
                           '-', ts.ndiascalendario,
                           '-', ts.nmostrar_selector) AS sdescripcion,
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS stipo_configuracion
             FROM tipos_solicitudes ts
             WHERE ts.nestado = 1
               and ts.nid_tipo_solicitud NOT IN (26)
               and ts.nid_categoria NOT IN (2))
            UNION
            (select ts.nid_tipo_solicitud::text AS scodigo,
                    CONCAT(ts.sdescripcion,
                           '-', ts.nid_categoria,
                           '-', ts.ntipo_frecuencia,
                           '-', ts.nrangomin,
                           '-',
                           CASE
                               WHEN ts.nid_tipo_solicitud in (3,10) THEN ROUND(
                                       svc.ddias_vencidos_laborales, 0)
                               WHEN ts.nid_tipo_solicitud in (22) THEN (TRUNC(svc.ddias_generados_laborales, 0))
                               END,
                           '-', ts.nrangominadm,
                           '-',
                           CASE
                               WHEN ts.nid_tipo_solicitud in (3,10) THEN ROUND(
                                       svc.ddias_vencidos_laborales, 0)
                               WHEN ts.nid_tipo_solicitud in (22) THEN (TRUNC(svc.dDias_Generados_Laborales, 0))
                               END,
                           '-', ts.narchivoobligatorio,
                           '-', ts.nafecta_asistencia,
                           '-', ts.nmostrar_calendario,
                           '-', ts.ntiempoanticipacion,
                           '-', ts.ndiascalendario,
                           '-', ts.nmostrar_selector) AS sdescripcion,
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS stipo_configuracion
             from tipos_solicitudes ts
                      join saldos_vacaciones_colaboradores_prueba svc
                           on svc.nid_colaborador = p_collaborator_id
             where ts.nestado = 1
               and ts.nid_categoria IN (2));
    end if;

    if 'revision' = any(var_configuration_request) then
        RETURN QUERY
            SELECT (select sfiltros
                    from revisiones
                    where nusuario_creador = p_user_id
                      and nestado = 1) AS scodigo,
                   'Reivision Activa' AS sdescripcion,
                   'REVISION'         AS stipo_configuracion;
    end if;

    if 'solicitudes_usuario' = any(var_configuration_request) then
        RETURN QUERY
            SELECT s.nid_solictud::text,
                   concat(ts.ntipo_frecuencia, '*', ts.nid_categoria, '*', s.nid_tipo_solicitud, '*', s.dfecha_inicio,
                          '*', s.dfecha_fin, '*', s.dhora_inicio, '*', s.dhora_fin),
                   'SOLICITUDES_USUARIO'
            FROM solicitudes s
            JOIN tipos_solicitudes ts on s.nid_tipo_solicitud = ts.nid_tipo_solicitud
            WHERE s.nid_colaborador = p_collaborator_id
              and ts.nid_categoria not in (4)
              and s.nestado_solicitud in (1, 3);
    end if;

    if 'prefijo_proyecto' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.scodigo,
                   c.sdescripcion,
                   c.stabla
            FROM configs c
            WHERE c.stabla = 'PREFIJO_PROYECTO';
    end if;

    if 'prefijo_requerimiento' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.scodigo,
                   c.sdescripcion,
                   c.stabla
            from configs c
            where c.stabla = 'PREFIJO_REQUERIMIENTO'
              AND scodigo = '1';
    end if;

    if 'solicitud_por_dias_mes' = any(var_configuration_request) then
        RETURN QUERY
            SELECT CASE
                       WHEN COUNT(s.nid_solictud) > 0 THEN '1'
                       ELSE '0'
                    END,
                   'SOLICTID_POR_DIAS',
                   'SOLICTID_POR_DIAS'
            FROM solicitudes s
            JOIN tipos_solicitudes ts ON ts.nid_tipo_solicitud = s.nid_tipo_solicitud
            WHERE s.nid_colaborador = p_collaborator_id
              AND ts.nafecta_asistencia = 1
              AND (ntipo_frecuencia = 2
                OR ntipo_frecuencia = 3)
              and s.nestado_solicitud = 3
              AND (NOW() AT TIME ZONE 'America/Lima')::date BETWEEN s.dfecha_inicio AND s.dfecha_fin;
    end if;

    if 'efectos_banner' = any(var_configuration_request) then
        RETURN QUERY
            SELECT NULL AS scodigo,
                   'Selecciona un efecto' AS sdescripcion,
                   'EFECTOS_BANNER' AS stipo_configuracion
            UNION
            SELECT scodigo AS scodigo,
                   sdescripcion AS sdescripcion,
                   'EFECTOS_BANNER' AS stipo_configuracion
            FROM configs
            WHERE stabla = 'EFECTOS_BANNER'
              AND nestado = 1;
    end if;

    if 'proyectos_lider' = any(var_configuration_request) then
        RETURN QUERY
            SELECT pl.nid_lider::text,
                   CONCAT(pl.nprincipal, ' ' + spersona_nombre),
                   'PRY_LIDERES'
            FROM proyecto_lider pl
            JOIN personas_colaboradores pc ON pc.nid_colaborador = pl.nid_lider
            WHERE pl.nestado = 1
              AND pl.nid_proyecto = p_project_id;
    end if;

    if 'coordinador_proyecto_requerimiento' = any(var_configuration_request) then
        RETURN QUERY
            SELECT '0',
                   CONCAT(cp.nid_coordinador, '-', cp.nid_proyecto, '-', r.nid_requerimiento),
                   'COORDINADOR_PROYECTO_REQUERIMIENTO'
            FROM requerimientos r
                     join coordinadores_proyectos cp on cp.nid_proyecto = r.nid_proyecto
                     join supervisor_proyecto sp on sp.nid_proyecto = cp.nid_proyecto
                     join coordinadores supervisor on supervisor.nid_coordinador = sp.nid_supervisor
                     join personas p_sup on p_sup.nid_persona = supervisor.nid_persona
                     join usuarios u_sup on u_sup.nid_persona = p_sup.nid_persona
            WHERE u_sup.nid_usuario = p_user_id;
    end if;

    if 'proyecto_requerimiento' = any(var_configuration_request) then
        RETURN QUERY
            SELECT '0',
                   CONCAT(cp.nid_proyecto, '-', r.nid_requerimiento),
                   'PROYECTO_REQUERIMIENTO'
            FROM requerimientos r
            JOIN coordinadores_proyectos cp on cp.nid_proyecto = r.nid_proyecto
            JOIN coordinadores coo on coo.nid_coordinador = cp.nid_coordinador
            JOIN personas p_sup on p_sup.nid_persona = coo.nid_persona
            JOIN usuarios u_sup on u_sup.nid_persona = p_sup.nid_persona
            WHERE u_sup.nid_usuario = p_user_id;
    end if;

    if 'todo_requerimiento' = any(var_configuration_request) then
        RETURN QUERY
            SELECT r.nid_requerimiento::text,
                   r.snombre,
                   'REQUERIMIENTOS_FILTRO'
            FROM requerimientos r;
    end if;

    if 'requerimiento_supervisor' = any(var_configuration_request) then
        RETURN QUERY
            select r.nid_requerimiento::text,
                   r.snombre,
                   'REQUERIMIENTOS_SUPERVISOR'
            from requerimientos r
                     JOIN proyectos pry ON pry.nid_proyecto = r.nid_proyecto
                     JOIN coordinadores_proyectos cp ON cp.nid_proyecto = pry.nid_proyecto
                -- join Coordinadores_Proyectos cp on cp.nId_Coordinador = c.nId_Coordinador
                -- JOIN Personas p ON c.nId_Persona = p.nId_Persona
                     join supervisor_proyecto sp on sp.nid_proyecto = cp.nid_proyecto
                     join coordinadores supervisor on supervisor.nid_coordinador = sp.nid_supervisor
                     join personas p_sup on p_sup.nid_persona = supervisor.nid_persona
                     join usuarios u_sup on u_sup.nid_persona = p_sup.nid_persona
            WHERE u_sup.nid_usuario = p_user_id
              AND pry.nestado = 1;
    end if;

    if 'requerimiento_coordinador' = any(var_configuration_request) then
        RETURN QUERY
            select r.nid_requerimiento::text,
                   r.snombre,
                   'REQUERIMIENTOS_COORDINADOR'
            from requerimientos r
            JOIN proyectos pry ON pry.nid_proyecto = r.nid_proyecto
            JOIN coordinadores_proyectos cp ON cp.nid_proyecto = pry.nid_proyecto
                -- join Coordinadores_Proyectos cp on cp.nId_Coordinador = c.nId_Coordinador
                -- JOIN Personas p ON c.nId_Persona = p.nId_Persona
                -- join Supervisor_Proyecto sp on sp.nId_Proyecto = cp.nId_Proyecto
            JOIN coordinadores coor on coor.nid_coordinador = cp.nid_coordinador
            JOIN personas p_coor on p_coor.nid_persona = coor.nid_persona
            JOIN usuarios u_coor on u_coor.nid_persona = p_coor.nid_persona
            WHERE u_coor.nid_usuario = p_user_id
              AND pry.nestado = 1;
    end if;

    if 'requerimientos_lider' = any(var_configuration_request) then
        if EXISTS (
            SELECT 1
            FROM proyecto_lider pl
            WHERE pl.nid_lider = p_collaborator_id
              AND pl.nprincipal = 1
              AND pl.nestado = 1
              AND pl.nid_proyecto = p_project_id
        ) then
            RETURN QUERY
                SELECT r.nid_requerimiento::text,
                       r.snombre,
                       'REQUERIMIENTOS_LIDER'
                FROM requerimientos r
                WHERE r.nid_proyecto = p_project_id and r.nestado = 1;
        else
            RETURN QUERY
            SELECT r.nid_requerimiento::text,
                   r.snombre,
                   'REQUERIMIENTOS_LIDER'
            FROM requerimientos r
            WHERE r.nId_Lider = p_collaborator_id
              AND r.nId_Proyecto = p_project_id and r.nestado = 1;
        end if;
    end if;

    if 'dias_laborales' = any(var_configuration_request) then
        RETURN QUERY
            SELECT '1',
                   STRING_AGG(hd.nDia, ', '),
                   'DIAS_LABORALES'
            FROM horarios_colaboradores hc
                     JOIN horarios_detalles hd ON hd.nid_horario = hc.nid_horario
                     JOIN personas_colaboradores pc ON pc.nid_colaborador = hc.nid_colaborador
                     JOIN usuarios u ON u.nid_persona = pc.nid_persona
            WHERE u.nid_usuario = p_user_id;
    end if;

    if 'rubros' = any(var_configuration_request) then
        RETURN QUERY
            SELECT r.nid_rubro::text,
                   r.sdescripcion,
                   'RUBRO'
            FROM rubros r
            WHERE r.nestado = 1;
    end if;

    /*if 'empresas_suggest' = any(var_configuration_request) then
        RETURN QUERY
            SELECT e.nid_empresa     AS sCodigo,
                   e.snombre_empresa AS sDescripcion,
                   'EMPRESAS'        AS sTipo_Configuracion
            FROM empresas e;
    end if;*/

    if 'monedas' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.scodigo,
                   CONCAT(c.sicono, '-', c.sdescripcion),
                   'MONEDAS'
            FROM configs c
            WHERE stabla = 'TIPO_MONEDA'
              AND nestado = 1;
    end if;

    if 'metodos_pago' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.scodigo,
                   c.sdescripcion,
                   'METODO_PAGO'
            FROM configs c
            WHERE stabla = 'METODO_PAGO'
              AND c.nestado = 1;
    end if;

    if 'estados solicitud tesoreria' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.scodigo,
                   c.sdescripcion,
                   'ESTADO_SOLICITUD_TESORERIA'
            FROM configs c
            WHERE stabla = 'ESTADO_SOLICITUD_TESORERIA'
              AND c.nestado = 1;
    end if;

    if 'client_company_suggest' = any(var_configuration_request) then
        RETURN QUERY
            SELECT nid_cliente::text,
                   snombre_cliente,
                   'COMPANY'
            FROM v_clients_list_mg;
    end if;

    if 'emails_list' = any(var_configuration_request) then
        RETURN QUERY
            SELECT nid_usuario::text,
                   semail,
                   'CORREO'
            FROM Usuarios
            WHERE nEstado = 1;
    end if;

    if 'solicitudes_pendientes' = any(var_configuration_request) then
        RETURN QUERY
            SELECT count(s.nid_solictud)::text,
                   null,
                   'SOLICITUDES_PENDIENTES_COLABORADOR'
            FROM solicitudes s
            WHERE s.nid_colaborador = p_collaborator_id
              and s.nestado_solicitud = 1;
    end if;

    if 'proyectos_total' = any(var_configuration_request) then
        RETURN QUERY
            SELECT COUNT(p.nid_proyecto)::text,
                   null,
                   'PROYECTOS_TOTAL'
            FROM proyectos p;
    end if;

    if 'acceso_proyectos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT c.scodigo,
                   c.sdescripcion,
                   'TIPO_ACCESO_PROYECTO'
            FROM configs c
            WHERE c.stabla = 'TIPO_ACCESO_PROYECTO'
            ORDER BY nordenamiento;
    end if;

    if 'plantillas_contratos' = any(var_configuration_request) then
        RETURN QUERY
            SELECT CONCAT(ct.nempresa_usuaria, '-', ct.stipo_contratacion),
                   ct.sid_drivefile,
                   'TEMPLATES'
            FROM contract_templates ct
            WHERE nestado = 1;
    end if;

    if 'proyectos_listado' = any(var_configuration_request) then
        if p_slug_listado = 'TODOS' then
            RETURN QUERY
                SELECT p.nid_proyecto::text,
                       p.snombre,
                       'PROYECTOS_LISTADO'
                FROM proyectos p;
        end if;

        if p_slug_listado = 'EQUIPO' then
            RETURN QUERY
                SELECT DISTINCT p.nid_proyecto::text,
                                p.snombre,
                                'PROYECTOS_LISTADO'
                FROM proyectos p
                         JOIN colaboradores_proyectos cp ON cp.nid_proyecto = p.nid_proyecto
                         JOIN team_colaboradors tc ON tc.nid_colaborador = cp.nid_colaborador
                         JOIN team t ON t.nid_team = tc.nid_team
                         JOIN personas_colaboradores pc ON pc.nid_colaborador = t.nid_lider
                         JOIN usuarios u ON u.nid_persona = pc.nid_persona
                WHERE u.nid_usuario = p_user_id;
        end if;

    end if;

    if 'cierre-tareo' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                nid_cierre::text,
                dfecha_cierre::text,
                'CIERRE_TAREO'
            FROM cierre_horas
            WHERE nestado_cierre = 1
                AND nestado = 1
            ORDER BY nId_Cierre DESC
            LIMIT 1;
    end if;

    if 'cierre-tareo-pendientes' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                nid_cierre::text,
                to_char(dfecha_cierre, 'MM-YYYY'),
                'CIERRE_TAREO_PENDIENTES'
            FROM
                cierre_horas
            WHERE
                nestado_cierre = 0
              AND nestado = 1
              AND dfecha_cierre <= (DATE_TRUNC('month', CURRENT_DATE + INTERVAL '2 months') - INTERVAL '1 day')::date
            ORDER BY
                nid_cierre;
    end if;

    if 'categorias_all' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                nid_categoria::text,
                snombre,
                'CATEGORIAS_ALL'
            FROM
                categorias
            WHERE
                nestado = 1;
    end if;

    if 'aprobadores' = any(var_configuration_request) then
        RETURN QUERY
            SELECT DISTINCT
                nid_aprobador::text,
                snombre_aprobador,
                'APROBADORES_SOLICITUDES'
            FROM v_listado_aprobadores_solicitudes
            ORDER BY nid_aprobador;
    end if;

    if 'emails_personal_groups' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                scodigo,
                sdescripcion,
                stipo_configuracion
            FROM v_listado_emails_personas;
    end if;

    if 'remitente' = any(var_configuration_request) then
        RETURN QUERY
            SELECT  DISTINCT
                nid_usuario::text,
                semail,
                'REMITENTE'
            from usuarios

            UNION
            SELECT  DISTINCT
                scodigo,
                sdescripcion,
                'REMITENTE'
            from configs WHERE nid_config = 328;
    end if;

    if 'localizacion_predeterminada' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                0,
                (
                    SELECT jsonb_agg(to_jsonb(c))
                    FROM (SELECT svalor_minimo AS latitude,
                                 sValor_Maximo AS longitude,
                                 0             AS accuracy
                          FROM configs
                          WHERE stabla = 'LOCALIZACION') c
                ),
                'LOCALIZACION_PREDETERMINADA' AS sTipo_Configuracion;
    end if;

    if 'modalidad_asistencia' = any(var_configuration_request) then

        var_modalidad := (SELECT fn_get_modalidad_colaborador(p_collaborator_id, p_slug_listado::timestamp));

        RETURN QUERY
            SELECT
                var_modalidad::text,
                sdescripcion,
                'MODALIDAD_ASISTENCIA'
            FROM configs
            WHERE stabla = 'TIPO_MODALIDAD'
              AND scodigo = var_modalidad::text;
    end if;

    if 'mailing_creator' = any(var_configuration_request) then
        RETURN QUERY
            select
                distinct
                mt.nusuario_creador::text,
                concat_ws(' ' ,p.sprimer_nombre, p.sape_paterno),
                'MAILING_CREATOR'
            from mailingtemplate mt
                     inner join usuarios s on s.nid_usuario = mt.nusuario_creador
                     inner join personas p on p.nid_persona = s.nid_persona;
    end if;

    if 'mailing_list' = any(var_configuration_request) then
        RETURN QUERY
            select
                nid_mailingtemplate::text,
                sname,
                'MAILING_LIST'
            from mailingtemplate;
    end if;

    if 'rango_marcas' = any(var_configuration_request) then
        RETURN QUERY
            select
                scodigo,
                sdescripcion,
                'RANGO_MARCAS'
            from configs c
            WHERE stabla = 'RANGO_MARCAS';
    end if;

    if 'evaluation_template' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                nid_form_template::text,
                sname,
                'EVALUATION_TEMPLATE'
            FROM form_template;
    end if;

    if 'evaluation' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                nid_evaluation::text,
                sname,
                'EVALUATION'
            FROM evaluations;
    end if;

    if 'form_state' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                scodigo,
                sdescripcion,
                'FORM_STATE'
            FROM configs
            WHERE stabla like('%ESTADO_FORMULARIO%');
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
                             WHEN (NOW() AT TIME ZONE 'America/Lima')::date BETWEEN e.dinit_date AND e.dend_date THEN 2
                             WHEN (NOW() AT TIME ZONE 'America/Lima')::date < e.dinit_date THEN 4
                             WHEN (NOW() AT TIME ZONE 'America/Lima')::date > e.dend_date THEN 3
                             END AS nstate -- cuando se filtre por estados
                     FROM evaluations e
                              JOIN forms f on e.nid_evaluation = f.nid_evaluation
                              JOIN evaluators ev on f.nid_form = ev.nid_form
                     WHERE ev.nid_evaluator = p_collaborator_id

                 ) e
            WHERE e.nstate NOT IN (1, 4)
            ORDER BY e.nid_evaluation DESC;
    end if;

    if 'forms' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                ft.nid_form_type::text,
                ft.sname,
                'FORMS'
            FROM form_type ft
            WHERE ft.nid_form_type NOT IN (
                SELECT distinct ft2.nid_formtype_padre
                FROM form_type ft2
                WHERE ft2.nid_formtype_padre IS NOT NULL
            ) AND ft.bstate = 1;
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
                             WHEN (NOW() AT TIME ZONE 'America/Lima')::date BETWEEN e.dinit_date AND e.dend_date THEN 2
                             WHEN (NOW() AT TIME ZONE 'America/Lima')::date < e.dinit_date THEN 4
                             WHEN (NOW() AT TIME ZONE 'America/Lima')::date > e.dend_date THEN 3
                             END AS nstate, -- cuando se filtre por estados
                         e.dshare_results
                     FROM evaluations e
                              JOIN forms f on e.nid_evaluation = f.nid_evaluation
                              JOIN evaluators ev on f.nid_form = ev.nid_form
                     WHERE ev.nid_evaluator = p_collaborator_id

                 ) e
            WHERE e.nstate = 3 and e.dshare_results IS NOT NULL and e.dshare_results <= (NOW() AT TIME ZONE 'America/Lima')::date
            ORDER BY e.nid_evaluation DESC;
    end if;

    if 'planned_closing' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
			    car.nid_cierre::text,
                car.dfecha_cierre_planificado,
                   'PLANNED_CLOSING'
            FROM cierreasistenciareportes car
            WHERE car.dfecha_cierre_planificado is not null
            ORDER BY car.nid_cierre DESC
            LIMIT 1;
    end if;

    if 'state_dashboard' = any(var_configuration_request) then
        RETURN QUERY
            SELECT
                c.scodigo,
                c.sdescripcion,
                'STATE_DASHBOARD'
            FROM configs c
            WHERE stabla = 'ESTADO_INDICADORES'
              AND nestado = 1;
    end if;

end;
$$;

alter function fn_get_configurations(text, integer, integer, integer, text, integer) owner to postgres;