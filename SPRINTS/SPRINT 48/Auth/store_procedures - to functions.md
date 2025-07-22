```sql
create function sp_getemail_login(par_semail text) returns SETOF user_login
    language plpgsql
as
$$
DECLARE
    var_Colaborador INTEGER;
    var_Coordinador INTEGER;
BEGIN
    /* Verificamos si el correo es perteneciente de un colaborador o coordinador */
    var_Colaborador := (SELECT
                            COUNT(*)
                        FROM tenant0001_dbo.usuarios AS u
                                 JOIN tenant0001_dbo.personas AS p
                                      ON p.nid_persona = u.nid_persona
                                 JOIN tenant0001_dbo.colaboradores AS c
                                      ON c.nid_persona = p.nid_persona
                        WHERE LOWER(u.semail) = LOWER(par_sEmail));
    var_Coordinador := (SELECT
                            COUNT(*)
                        FROM tenant0001_dbo.usuarios AS u
                                 JOIN tenant0001_dbo.personas AS p
                                      ON p.nid_persona = u.nid_persona
                                 JOIN tenant0001_dbo.coordinadores AS c
                                      ON c.nid_persona = p.nid_persona
                        WHERE LOWER(u.semail) = LOWER(par_sEmail));

    IF var_Colaborador > 0 THEN
        RETURN QUERY
            SELECT
                u.nid_usuario,
                c.nid_colaborador,
                r.sdescripcion AS snombre_rol,
                u.semail,
                u.sclave,
                s.stoken_usuario AS stoken,
                p.spersona_nombre,
                CONCAT(p.sprimer_nombre, ' ', p.sape_paterno) AS snombres,
                p.sape_paterno,
                p.sape_materno,
                p.dfecha_nacimiento,
                ru.nid_rol,
                c2.nid_cargo,
                u.nestado AS nestado_usuario,
                c.nestado_colaborador AS nestado_colaborador,
                u.ntipo_autenticacion
            FROM tenant0001_dbo.usuarios AS u
                     JOIN tenant0001_dbo.personas AS p
                          ON p.nid_persona = u.nid_persona
                     JOIN tenant0001_dbo.colaboradores AS c
                          ON c.nid_persona = p.nid_persona
                     JOIN tenant0001_dbo.roles_usuarios AS ru
                          ON ru.nid_usuario = u.nid_usuario
                     JOIN tenant0001_dbo.roles AS r
                          ON ru.nid_rol = r.nid_rol
                     JOIN tenant0001_dbo.cargos_colaboradores AS cc
                          ON cc.nid_colaborador = c.nid_colaborador
                     JOIN tenant0001_dbo.cargos AS c2
                          ON c2.nid_cargo = cc.nid_cargo
                     LEFT OUTER JOIN tenant0001_dbo.sesiones AS s
                                     ON s.nid_usuario = u.nid_usuario
            WHERE LOWER(u.semail) = LOWER(par_sEmail) AND u.nestado = 1 AND cc.nestado = 1;
    END IF;

    IF var_Coordinador > 0 THEN
        RETURN QUERY
            SELECT
                u.nid_usuario, c.nid_coordinador AS nid_colaborador, r.sdescripcion AS snombre_rol, u.semail, u.sclave, s.stoken_usuario AS stoken, p.spersona_nombre, CONCAT(p.sprimer_nombre, ' ', p.sape_paterno) AS snombres, p.sape_paterno, p.sape_materno, p.dfecha_nacimiento, ru.nid_rol, 0 AS nid_cargo, u.nestado AS nestado_usuario, c.nestado AS nestado_colaborador, u.ntipo_autenticacion
            FROM tenant0001_dbo.usuarios AS u
                     JOIN tenant0001_dbo.personas AS p
                          ON u.nid_persona = p.nid_persona
                     JOIN tenant0001_dbo.coordinadores AS c
                          ON c.nid_persona = p.nid_persona
                     JOIN tenant0001_dbo.roles_usuarios AS ru
                          ON ru.nid_usuario = u.nid_usuario
                     JOIN tenant0001_dbo.roles AS r
                          ON r.nid_rol = ru.nid_rol
                     LEFT OUTER JOIN tenant0001_dbo.sesiones AS s
                                     ON s.nid_usuario = u.nid_usuario
            WHERE LOWER(u.semail) = LOWER(par_sEmail) AND u.nestado = 1;
    END IF;
END;
$$;

--corregir
create function sp_smart_get_email_login_super(par_semail text) returns setof user_login
    language plpgsql
as
$$
BEGIN
        /* Recuperar los datos del usuario master */
    RETURN QUERY
    SELECT
        u.nid_usuario,
        NULL AS nid_colaborador,
        r.sdescripcion AS snombre_rol,
        par_semail AS semail,
        u.sclave AS sclave,
        s.stoken_usuario AS stoken_usuario,
        p.spersona_nombre AS spersona_nombre,
        CONCAT(p.sprimer_nombre, ' ', p.sape_paterno) AS snombres,
        p.sape_paterno AS sape_paterno,
        p.sape_materno AS sape_materno,
        p.dfecha_nacimiento AS dfecha_nacimiento,
        r.nid_rol AS nid_rol,
        0 AS nid_cargo,
        u.nestado AS nestado_usuario,
        NULL AS nestado_colaborador,
        u.ntipo_autenticacion AS ntipo_autenticacion
    INTO result
    FROM tenant0001_dbo.usuarios AS u
             JOIN tenant0001_dbo.personas AS p
                  ON u.nid_persona = p.nid_persona
             JOIN tenant0001_dbo.roles_usuarios AS ru
                  ON ru.nid_usuario = u.nid_usuario
             JOIN tenant0001_dbo.roles AS r
                  ON r.nid_rol = ru.nid_rol
             LEFT OUTER JOIN tenant0001_dbo.sesiones AS s
                             ON s.nid_usuario = u.nid_usuario
    WHERE LOWER(u.semail) = LOWER(par_semail);
END;
$$;


create function sp_smart_get_session_by_user(p_nid_user int) RETURNS SETOF sesiones
    language plpgsql
as
$$
BEGIN
    RETURN QUERY
    SELECT
        s.nid_usuario,
        s.stoken_usuario,
        s.nip,
        s.nso,
        s.ndispositivo,
        s.nnavegador,
        s.nestado,
        s.nusuario_creador,
        s.ddatetime_creador,
        s.nusuario_update,
        s.ddatetime_update,
        s.nusuario_delete,
        s.ddatetime_delete
    FROM sesiones s
    WHERE s.nid_usuario = p_nid_user;

END
$$