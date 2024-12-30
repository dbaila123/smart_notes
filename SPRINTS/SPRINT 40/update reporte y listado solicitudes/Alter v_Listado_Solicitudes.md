```sql
CREATE OR ALTER VIEW [dbo].[v_Listado_Solicitudes] AS    
SELECT    
  tb_soli.nId_Solictud nId,    
  u_owner.nId_Colaborador nId_Colaborador,    
  tb_soli.nId_Tarea nId_Tarea,    
  u_owner.sUrl_Foto sUrl_Foto,    
  u_owner.sPersona_Nombre sNombre_Colaborador,  
  docs.sTipo_Documento as sTipo_Documento,     
  docs.sNumero_Documento,  
 CONCAT( docs.sTipo_Documento,     ' ',     docs.sNumero_Documento ) as sDocumento_Completo,  
  (CASE    
   WHEN tb_soli.nId_Tarea IS NOT NULL    
    THEN CONVERT(varchar, Tareas.dDatetime_Creador, 120)    
    ELSE CONVERT(varchar, tb_soli.dDatetime_Creador, 120)    
  END) dFecha_Solicitud,    
  tb_soli.nId_Tipo_Solicitud nId_Tipo_Solicitud,    
  tb_ts.sDescripcion sNombre_Tipo_Solicitud,    
  CONCAT(CONVERT(varchar, tb_soli.dFecha_Inicio), ' ', CONVERT(varchar, tb_soli.dHora_Inicio)) dFecha_Inicio,    
  CONCAT(CONVERT(varchar, tb_soli.dFecha_Fin), ' ', CONVERT(varchar, tb_soli.dHora_Fin)) dFecha_Fin,    
  tb_soli.nCantidad nCantidad,    
  u_owner.nTipo_Modalidad nTipo_Modalidad,    
  tb_conf_modalidad.sDescripcion sNombre_Modalidad,    
  tb_campus.nId_Sede nSede,    
  tb_campus.sDescripcion sNombre_Sede,    
  tb_soli.nUsuario_Update nUsuario_Update,    
  ISNULL(sh.nTotal_sh, 1) AS nFlag_Historico,    
  (CASE    
    WHEN    
      tb_soli.nUsuario_Update IS NULL THEN (    
        CASE    
            WHEN    
            (tb_soli.nId_Tipo_Solicitud = 9 AND    
            tb_soli.nEstado_Solicitud = 3) OR    
            (tb_soli.nUsuario_Creador != u_owner.nId_Usuario) THEN 'AUTOMÁTICO'    
            ELSE ''    
        END    
        )    
    ELSE (    
            SELECT TOP 1    
                CONCAT(tb_person_update.sPrimer_Nombre, ' ', tb_person_update.sApe_Paterno)    
            FROM Usuarios tb_user_update    
            JOIN Personas tb_person_update    
                ON tb_user_update.nId_Persona = tb_person_update.nId_Persona    
            WHERE tb_user_update.nId_Usuario = tb_soli.nUsuario_Update    
        --     )    
        -- END    
    )    
  END) sNombre_Usuario_Update,    
  --APROBADOR    
  (    
    CASE    
      -- extra hours    
      WHEN (tb_soli.nId_Tipo_Solicitud = 9 AND tb_soli.nEstado_Solicitud = 3) THEN '[]'    
      -- is created by rrhh    
      WHEN (tb_soli.nUsuario_Creador != u_owner.nId_Usuario) THEN '[]'    
      ------------------------------------------------------------------------------------------>    
      ELSE tb_soli.sAprobadores    
    --   ELSE '[]'    
    END    
  ) aprobadores,    
  ------------------------------------------------------------------------------------------------>    
  CONVERT(INT,tb_soli.nEstado_Solicitud) nEstado,    
  tb_conf_estado.sDescripcion sEstado_Solicitud,    
  --DETALLE    
  (CASE    
    WHEN    
      tb_soli.nId_Proyecto = NULL THEN 0    
    ELSE tb_soli.nId_Proyecto    
  END) nId_Proyecto,    
    p.sNombre as sNombre_Proyecto,    
 req.sNombre as sNombre_Requerimiento,    
 cat.sNombre as sNombre_Categoria_Requerimiento,    
  tm.nId_Tipo_Motivo nId_Tipo_Motivo,    
  tm.sDescripcion sNombre_Tipo_Motivo,    
  tb_soli.sMotivo sMotivo,    
  tb_conf_modalidad.sIcono sIcono_Tipo_Modalidad,    
  CASE    
    WHEN u_owner.nId_Usuario = tb_soli.nUsuario_Creador THEN NULL    
    ELSE CONCAT(creator.sPrimer_Nombre, ' ', creator.sApe_Paterno)    
  END sPersona_Nombre_Creator,    
  tb_ts.nId_Categoria,    
  tb_ts.nTipo_Frecuencia,    
  CONCAT(psup.sPrimer_Nombre, ' ', psup.sApe_Paterno) AS sNombre_Lider,    
  cs.nId_Supervisor AS nId_Lider    
FROM Solicitudes tb_soli  
OUTER APPLY (     SELECT TOP 1          d.sNumero_Documento,         con.sDescripcion as sTipo_Documento     FROM Colaboradores c      INNER JOIN Personas p ON c.nId_Persona = p.nId_Persona      INNER JOIN Documentos d ON p.nId_Persona = d.nId_Persona    
 INNER JOIN Configs con ON d.nTipo_Documento = con.sCodigo          AND con.sTabla = 'TIPO_DOCUMENTO'     WHERE c.nId_Colaborador = tb_soli.nId_Colaborador ) docs  
--------- Owner    
JOIN vw_user_person_collab u_owner    
  ON tb_soli.nId_Colaborador = u_owner.nId_Colaborador    
  ----------------------    
JOIN Tipos_Solicitudes tb_ts    
  ON tb_soli.nId_Tipo_Solicitud = tb_ts.nId_Tipo_Solicitud    
JOIN Configs tb_conf_modalidad    
  ON CAST(u_owner.nTipo_Modalidad as VARCHAR(10)) = tb_conf_modalidad.sCodigo    
  AND tb_conf_modalidad.sTabla = 'TIPO_MODALIDAD'    
JOIN Sedes_Colaboradores tb_sc    
  ON u_owner.nId_Colaborador = tb_sc.nId_Colaborador    
JOIN Sedes tb_campus    
  ON tb_sc.nId_Sede = tb_campus.nId_Sede    
JOIN Configs tb_conf_estado    
  ON CAST(tb_soli.nEstado_Solicitud as VARCHAR(10)) = tb_conf_estado.sCodigo    
  AND tb_conf_estado.sTabla = 'ESTADO_SOLICITUD'    
JOIN Tipos_Motivos tm    
  ON tm.nId_Tipo_Motivo = tb_soli.nId_Tipo_Motivo    
JOIN colaboradores_supervisor cs ON cs.nId_Colaborador = tb_soli.nId_Colaborador     
  AND cs.bPrincipal = 1    
JOIN Colaboradores csup ON cs.nId_Supervisor = csup.nId_Colaborador    
JOIN Personas psup ON psup.nId_Persona = csup.nId_Persona     
  --proyecto / requerimietno / categorias    
LEFT JOIN Tareas tareas    
  ON tb_soli.nId_Tarea = tareas.nId_Tarea    
LEFT JOIN Proyectos p    
  ON tb_soli.nId_Proyecto = p.nId_Proyecto    
LEFT JOIN Requerimientos req    
  ON tareas.nId_Requerimiento = req.nId_Requerimiento    
LEFT JOIN Categorias cat    
  ON CAST(tareas.sId_Categoria AS INT) = cat.nId_Categoria    
    
LEFT JOIN (SELECT COUNT(*) AS nTotal_sh, nId_Solicitud FROM v_Listado_Historico_Solicitudes    
GROUP BY nId_Solicitud) sh ON sh.nId_Solicitud = tb_soli.nId_Solictud    
JOIN vw_user_person_collab creator    
ON creator.nId_Usuario = tb_soli.nUsuario_Creador    
WHERE tb_soli.nEstado = 1; 
```

