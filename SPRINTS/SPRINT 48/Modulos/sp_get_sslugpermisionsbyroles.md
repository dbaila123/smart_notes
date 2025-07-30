```POSTGRESQL
create or replace function fn_get_sslugpermisionsbyroles(p_idmodulo integer, p_nestadooption integer, p_idrol integer, p_nestadop integer, p_nid_colaborador integer DEFAULT NULL::integer)  
    returns TABLE(sslug text, sdecripcion text)  
    language plpgsql  
as  
$$  
BEGIN  
    -- Crear tabla temporal para almacenar los permisos  
    CREATE TEMP TABLE IF NOT EXISTS temp_sPermisions (  
        sSlug TEXT,  
        sDecripcion TEXT  
    );  
  
    -- Limpiar la tabla temporal por si existe  
    DELETE FROM temp_sPermisions;  
  
    -- Insertar permisos basados en rol y opciones  
    INSERT INTO temp_sPermisions (sSlug, sDecripcion)  
    SELECT o.sSlug, o.sDescripcion  
    FROM Opciones o  
    INNER JOIN Permisos_Opciones po ON o.nId_Opcion = po.nId_Opcion  
    WHERE po.nId_Rol = p_IdRol  
      AND o.nEstado = p_nEstadoOption  
      AND po.nEstado = p_nEstadoP  
      AND o.nId_Modulo = p_IdModulo;  
  
    -- Verificar si el colaborador tiene subordinados  
    IF p_nId_Colaborador IS NOT NULL AND  
       (SELECT COUNT(*) FROM colaboradores_supervisor WHERE nId_Supervisor = p_nId_Colaborador) > 0 THEN  
  
        INSERT INTO temp_sPermisions (sSlug, sDecripcion)  
        VALUES ('TEAM-HAVE-SUBORDINATE', 'VER LISTADO DE EQUIPO');  
  
    END IF;  
  
    -- Retornar todos los permisos  
    RETURN QUERY SELECT temp_sPermisions.sSlug, temp_sPermisions.sDecripcion FROM temp_sPermisions;  
  
    -- Limpiar tabla temporal  
    DROP TABLE IF EXISTS temp_sPermisions;  
  
END;  
$$;
```