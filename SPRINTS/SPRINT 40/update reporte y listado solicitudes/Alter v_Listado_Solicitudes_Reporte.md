```sql
-- dbo.v_Listado_Solicitudes_Reporte source      
      
CREATE OR ALTER VIEW [dbo].[v_Listado_Solicitudes_Reporte] AS      
SELECT      
  tb_soli.nId_Solictud nId,      
  u_owner.nId_Colaborador nId_Colaborador,    
 -- CONCAT(    
 --CASE          WHEN docs.sTipo_Documento = 'PASAPORTE' THEN 'PAS'         ELSE docs.sTipo_Documento     END, '  ', docs.sNumero_Documento    
 -- ) as sDocumento_Completo,    
  docs.sTipo_Documento as sTipo_Documento,  
  docs.sNumero_Documento,  
  tb_soli.nId_Tarea,      
  aprob.sAprobadores AS sAprobadores,      
  tb_soli.nId_Proyecto nId_Proyecto,      
  tb_ts.nId_Categoria nId_Categoria,      
  tb_soli.nId_Tipo_Solicitud nId_Tipo_Solicitud,      
  u_owner.sPersona_Nombre sNombre_Colaborador,      
  ISNULL(sh.nTotal_sh, 0) AS nFlag_Historico,      
    (CASE       
    WHEN tb_soli.nId_Tarea IS NOT NULL      
      THEN CONVERT(varchar, vt.dDatetime_Creador, 120)      
     ELSE CONVERT(varchar, tb_soli.dDatetime_Creador, 120)      
   END) dFecha_Solicitud,      
  CONCAT(CONVERT(varchar, tb_soli.dFecha_Inicio), ' ', CONVERT(varchar, tb_soli.dHora_Inicio)) dFecha_Inicio,      
  CONCAT(CONVERT(varchar, tb_soli.dFecha_Fin), ' ', CONVERT(varchar, tb_soli.dHora_Fin)) dFecha_Fin,      
  CONVERT(INT,tb_soli.nEstado_Solicitud) nEstado,      
  ------------------------------------------------------------------------------------------->      
  tb_ts.sDescripcion sNombre_Tipo_Solicitud,      
  case      
   when (select ts.nTipo_Frecuencia from Tipos_Solicitudes ts where ts.nId_Tipo_Solicitud = tb_soli.nId_Tipo_Solicitud) = 1 then 100*(tb_soli.nCantidad/60)/100      
   else tb_soli.nCantidad      
  end nCantidad_Reporte,      
  case      
   when (select ts.nTipo_Frecuencia from Tipos_Solicitudes ts where ts.nId_Tipo_Solicitud = tb_soli.nId_Tipo_Solicitud) = 1 then 'Hora(s)'      
   when (select ts.nTipo_Frecuencia from Tipos_Solicitudes ts where ts.nId_Tipo_Solicitud = tb_soli.nId_Tipo_Solicitud) = 2 then 'Día(s)'      
   when (select ts.nTipo_Frecuencia from Tipos_Solicitudes ts where ts.nId_Tipo_Solicitud = tb_soli.nId_Tipo_Solicitud) = 3 then 'Mes(es)'      
   else '-'      
  end sTipo_Cantidad,      
  tb_campus.sDescripcion sNombre_Sede,      
  (CASE      
    WHEN      
      tb_soli.nUsuario_Update IS NULL THEN (CASE      
        WHEN      
          (tb_soli.nId_Tipo_Solicitud = 9 AND      
          tb_soli.nEstado_Solicitud = 3) OR      
          (tb_soli.nUsuario_Creador != u_owner.nId_Usuario) THEN 'AUTOMÁTICO'      
        ELSE ''      
      END)      
    ELSE (CASE      
        WHEN      
          tb_soli.nUsuario_Update = u_owner.nId_Usuario THEN ''      
        ELSE (SELECT TOP 1      
            CONCAT(tb_person_update.sPrimer_Nombre, ' ', tb_person_update.sApe_Paterno)      
          FROM Usuarios tb_user_update      
          JOIN Personas tb_person_update      
            ON tb_user_update.nId_Persona = tb_person_update.nId_Persona      
          WHERE tb_user_update.nId_Usuario = tb_soli.nUsuario_Update)      
      END)      
  END) sNombre_Usuario_Update,      
  tb_conf_estado.sDescripcion sEstado_Solicitud,      
  vt.sEstado_Pago,      
  vt.dFecha_Cierre,      
  tb_soli.sMotivo sMotivo,      
  CONCAT(psup.sPrimer_Nombre, ' ', psup.sApe_Paterno) AS sNombre_Lider      
FROM Solicitudes tb_soli      
LEFT JOIN (SELECT       
        nId_Solictud,       
        CASE      
            WHEN sAprobadores = '[]' OR sAprobadores = 'END' THEN NULL      
            ELSE STRING_AGG(JSON_VALUE(value, '$.name'), ', ')      
        END AS sAprobadores       
    FROM Solicitudes        
    CROSS APPLY OPENJSON(CASE       
                            WHEN sAprobadores = '[]' OR sAprobadores = 'END' THEN NULL      
                            ELSE sAprobadores       
                        END)      
    WHERE sAprobadores IS NOT NULL AND sAprobadores NOT IN ('[]', 'END')      
    GROUP BY nId_Solictud, sAprobadores) aprob ON aprob.nId_Solictud = tb_soli.nId_Solictud      
JOIN vw_user_person_collab u_owner      
  ON tb_soli.nId_Colaborador = u_owner.nId_Colaborador    
  OUTER APPLY (     SELECT TOP 1          d.sNumero_Documento,         con.sDescripcion as sTipo_Documento     FROM Colaboradores c      INNER JOIN Personas p ON c.nId_Persona = p.nId_Persona     INNER JOIN Documentos d ON p.nId_Persona = d.nId_Persona   
  
  INNER JOIN Configs con ON d.nTipo_Documento = con.sCodigo          AND con.sTabla = 'TIPO_DOCUMENTO'     WHERE c.nId_Colaborador = u_owner.nId_Colaborador ) docs    
JOIN Tipos_Solicitudes tb_ts      
  ON tb_soli.nId_Tipo_Solicitud = tb_ts.nId_Tipo_Solicitud      
JOIN Sedes_Colaboradores tb_sc      
  ON u_owner.nId_Colaborador = tb_sc.nId_Colaborador      
JOIN Sedes tb_campus      
  ON tb_sc.nId_Sede = tb_campus.nId_Sede      
JOIN Configs tb_conf_estado      
  ON CAST(tb_soli.nEstado_Solicitud as VARCHAR(10)) = tb_conf_estado.sCodigo      
  AND tb_conf_estado.sTabla = 'ESTADO_SOLICITUD'      
  JOIN colaboradores_supervisor cs ON cs.nId_Colaborador = tb_soli.nId_Colaborador       
  AND cs.bPrincipal = 1      
JOIN Colaboradores csup ON cs.nId_Supervisor = csup.nId_Colaborador      
JOIN Personas psup ON psup.nId_Persona = csup.nId_Persona       
LEFT JOIN v_Listado_Tareas_Reporte vt ON vt.nId = tb_soli.nId_Tarea       
LEFT JOIN (SELECT COUNT(*) AS nTotal_sh, nId_Solicitud FROM v_Listado_Historico_Solicitudes       
GROUP BY nId_Solicitud) sh ON sh.nId_Solicitud = tb_soli.nId_Solictud      
WHERE tb_soli.nEstado = 1;
```


