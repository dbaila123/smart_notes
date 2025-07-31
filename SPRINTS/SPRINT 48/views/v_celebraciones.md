```SQL
CREATE OR REPLACE VIEW v_celebraciones AS
SELECT 
    nid_persona,
    nid_colaborador,
    nid_usuario,
    spersona_nombre,
    snombre_corto,
    CASE
        WHEN ncodigo_estado = 1 THEN 
            dfecha_nacimiento + INTERVAL '1 year' * (EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM dfecha_nacimiento))
        ELSE 
            dfecha_ingreso + INTERVAL '1 year' * (EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM dfecha_ingreso))
    END AS dfecha_celebration,
    dfecha_nacimiento,
    dfecha_ingreso,
    stelefono,
    ncodigo_tipo_modalidad,
    surl_foto,
    stipo_modalidad,
    ncodigo_sede,
    ssede,
    ncodigo_estado,
    sestado,
    nyears
FROM (
    SELECT
        p.nid_persona,
        c.nid_colaborador,
        u.nid_usuario,
        p.spersona_nombre,
        p.sprimer_nombre || ' ' || p.sape_paterno AS snombre_corto,
        p.dfecha_nacimiento,
        c.dfec_ingreso::date AS dfecha_ingreso,
        t.stelefono,
        c.ntipo_modalidad AS ncodigo_tipo_modalidad,
        c.surl_foto,
        m.sdescripcion AS stipo_modalidad,
        s.nid_sede AS ncodigo_sede,
        s.sdescripcion AS ssede,
        2 AS ncodigo_estado,
        'ANIVERSARIO' AS sestado,
        EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM c.dfec_ingreso) AS nyears
    FROM 
        colaboradores c
        JOIN personas p ON c.nid_persona = p.nid_persona
        JOIN usuarios u ON p.nid_persona = u.nid_persona
        JOIN roles_usuarios ru ON ru.nid_usuario = u.nid_usuario
        JOIN configs m ON c.ntipo_modalidad::text = m.scodigo AND m.stabla = 'TIPO_MODALIDAD'
        JOIN telefonos t ON p.nid_persona = t.nid_persona AND t.nprincipal = 1
        JOIN sedes_colaboradores sc ON c.nid_colaborador = sc.nid_colaborador
        JOIN sedes s ON sc.nid_sede = s.nid_sede
    WHERE 
        c.nestado_colaborador = 1
        AND EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM c.dfec_ingreso) > 0
        AND p.dfecha_nacimiento IS NOT NULL
        AND c.dfec_ingreso IS NOT NULL
        AND ru.nid_rol != 15
    UNION
    SELECT
        p.nid_persona,
        c.nid_colaborador,
        u.nid_usuario,
        p.spersona_nombre,
        p.sprimer_nombre || ' ' || p.sape_paterno AS snombre_corto,
        p.dfecha_nacimiento,
        c.dfec_ingreso::date AS dfecha_ingreso,
        t.stelefono,
        c.ntipo_modalidad AS ncodigo_tipo_modalidad,
        c.surl_foto,
        m.sdescripcion AS stipo_modalidad,
        s.nid_sede AS ncodigo_sede,
        s.sdescripcion AS ssede,
        1 AS ncodigo_estado,
        'CUMPLEAÑOS' AS sestado,
        EXTRACT(YEAR FROM CURRENT_DATE) - EXTRACT(YEAR FROM p.dfecha_nacimiento) AS nyears
    FROM 
        colaboradores c
        JOIN personas p ON c.nid_persona = p.nid_persona
        JOIN usuarios u ON p.nid_persona = u.nid_persona
        JOIN roles_usuarios ru ON ru.nid_usuario = u.nid_usuario
        JOIN configs m ON c.ntipo_modalidad::text = m.scodigo AND m.stabla = 'TIPO_MODALIDAD'
        JOIN telefonos t ON p.nid_persona = t.nid_persona AND t.nprincipal = 1
        JOIN sedes_colaboradores sc ON c.nid_colaborador = sc.nid_colaborador
        JOIN sedes s ON sc.nid_sede = s.nid_sede
    WHERE 
        c.nestado_colaborador = 1
        AND p.dfecha_nacimiento IS NOT NULL
        AND c.dfec_ingreso IS NOT NULL
        AND ru.nid_rol != 15
        AND DATE_PART('year', CURRENT_DATE) - DATE_PART('year', p.dfecha_nacimiento) > 0
) AS temp_celebraciones;