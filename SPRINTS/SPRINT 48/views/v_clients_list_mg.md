```sql
CREATE VIEW v_clients_list_mg AS
SELECT
    cli.nid_cliente,
    cli.nestado AS nestado_cliente,
    config.sdescripcion AS sestado_cliente,
    p_juridica.spersona_nombre AS snombre_cliente,
    doc.snumero_documento AS sruc,
    CONCAT(direcc.scalle, ', ', ubigeo.subigeo_distrito, ', ', ubigeo.subigeo_departamento, ', ', pais.sdescripcion) AS sdireccion_completa,
    --todo: check why this is not working
    -- cli.dDateTime_Creador AS dFecha_Creacion,
    cli.ddatetime_creador AS dfecha_creacion,
    -- Direction
    ubigeo.nid_ubigeo::text AS scodigo_ubigeo,
    ubigeo.subigeo_departamento                         AS sdepartamento,
    ubigeo.subigeo_provincia                            AS sprovincia,
    ubigeo.subigeo_distrito                             AS sdistrito,
    direcc.scalle                                       AS sdireccion,
    direcc.scalle2                                      AS sreferencia,

    -- contact information
    p_contacto.nid_persona AS nid_persona_contacto,
    p_contacto.spersona_nombre AS snombre_contacto,
    email.semail,
    -- Phone
    tel.stelefono AS stelefono,
    tel.sprefijo_area_local AS sprefijo_area_local,
    tel.nid_pais::text AS scodigo_pais,
    tel.ntipo_telefono::text AS scodigo_tipo_telefono

FROM clientes cli
         JOIN configs config ON  config.scodigo = cli.nestado::text AND  config.stabla = 'ESTADO'
         JOIN personas p_juridica ON p_juridica.nid_persona = cli.nid_persona
         JOIN documentos doc ON doc.nid_persona = p_juridica.nid_persona
         JOIN direcciones direcc ON direcc.nid_persona = p_juridica.nid_persona
         JOIN ubigeos ubigeo ON ubigeo.nid_ubigeo = direcc.nid_ubigeo
         JOIN paises pais ON pais.nid_pais = ubigeo.nid_pais

-- natural person contact
         LEFT JOIN contactos ON contactos.nid_persona = p_juridica.nid_persona AND contactos.nestado = 1
         LEFT JOIN personas p_contacto ON p_contacto.nId_Persona = contactos.nid_persona_contacto AND p_contacto.nestado = 1
         LEFT JOIN telefonos tel ON tel.nid_persona = contactos.nid_persona_contacto AND tel.nestado = 1
         LEFT JOIN emails email ON email.nid_persona = contactos.nid_persona_contacto AND email.nestado = 1

