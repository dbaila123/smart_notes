```POSTGRESQL
create oe replace function fn_get_modulos_rol(p_nid_rol integer)
    returns TABLE(nid_modulo integer, sdescripcion character varying, sicono character varying, npadre integer, surl_link_modulo character varying, norden integer, bisconfigoption integer, slabel character varying, bmostrar numeric)
    language sql
as
$$
    SELECT DISTINCT
        m1.nid_modulo,
        m1.sdescripcion,
        m1.sIcono,
        m1.nPadre,
        m1.sUrl_Link_Modulo,
        m1.nOrden,
        COALESCE(m1.nEstado_Config, 0) AS bIsConfigOption,
        CASE
            WHEN m1.sdescripcion = 'Configuración' THEN 'Configuraciones rápidas'
            WHEN m1.sdescripcion = 'Roles y accesos' THEN 'Roles'
            ELSE m1.sdescripcion
        END AS sLabel,
        v_Mostrar AS bMostrar
    FROM modulos m1,
    (
        SELECT
            mp.nId_Modulo AS npadre,
            mp.sDescripcion AS dmodulop,
            m.nId_Modulo,
            m.sDescripcion AS dmodulo,
            p.nId_Opcion,
            o.sDescripcion AS dopcion,
            surl_link_formulario,
            p.sIcono
        FROM permisos_opciones p
        INNER JOIN opciones o ON o.nid_opcion = p.nid_opcion
        INNER JOIN modulos m ON m.nid_modulo = o.nId_Modulo
        LEFT JOIN modulos mp ON mp.nId_Modulo = m.nPadre
        WHERE p.nid_rol = p_nid_rol
        AND p.nestado = 1
    ) bb
    WHERE m1.nEstado = 1
    AND (m1.nid_modulo = bb.nid_modulo OR m1.nid_modulo = bb.npadre);
$$;
```