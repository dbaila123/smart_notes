```sql
IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'categorias_solicitudes') = 1    
        BEGIN    
            INSERT INTO @configuraciones    
            SELECT cp.nId_Categoria         AS sCodigo,    
                   cp.sNombre               AS sDescripcion,    
                   'CATEGORIAS_SOLICITUDES' AS sTipo_Configuracion    
            FROM Categorias_Permisos cp    
            WHERE cp.nEstado = 1    
            UNION    
            (SELECT ts.nId_Tipo_Solicitud             AS sCodigo,    
                    CONCAT(ts.sDescripcion,    
                           '-', ts.nId_Categoria,    
                           '-', ts.nTipo_Frecuencia,    
                           '-', ts.nRangoMin,    
                           '-', ts.nRangoMax,    
                           '-', ts.nRangoMinAdm,    
                           '-', ts.nRangoMaxAdm,    
                           '-', ts.nArchivoObligatorio,    
                           '-', ts.nAfecta_Asistencia,    
                           '-', ts.nMostrar_Calendario,    
                           '-', ts.nTiempoAnticipacion,    
                  '-', ts.nDiasCalendario,    
           '-', ts.nMostrar_Selector) AS sDescripcion,    
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS sTipo_Configuracion    
             FROM Tipos_Solicitudes ts    
             WHERE ts.nEstado = 1    
               and ts.nId_Categoria NOT IN (2)
			   and ts.nId_Tipo_Solicitud NOT IN(26)
			   )
			   
            UNION    
            (select ts.nId_Tipo_Solicitud             AS sCodigo,    
                    CONCAT(ts.sDescripcion,    
       '-', ts.nId_Categoria,    
                           '-', ts.nTipo_Frecuencia,    
                           '-', ts.nRangoMin,    
                           '-',    
                           CASE    
                               WHEN ts.nId_Tipo_Solicitud in (3, 10) THEN ROUND(    
                                       svc.dDias_Vencidos_Laborales + svc.dDias_Ganados_Laborales, 0, 0)    
                               WHEN ts.nId_Tipo_Solicitud in (22) THEN (ROUND(svc.dDias_Generados_Laborales, 0, -1))    
                             END,    
                           '-', ts.nRangoMinAdm,    
  '-',    
       CASE    
  WHEN ts.nId_Tipo_Solicitud in (3, 10) THEN ROUND(    
                               svc.dDias_Vencidos_Laborales + svc.dDias_Ganados_Laborales, 0, 0)    
                               WHEN ts.nId_Tipo_Solicitud in (22) THEN (ROUND(svc.dDias_Generados_Laborales, 0, -1))    
                          END,    
                           '-', ts.nArchivoObligatorio,    
               '-', ts.nAfecta_Asistencia,    
                           '-', ts.nMostrar_Calendario,    
                        '-', ts.nTiempoAnticipacion,    
                           '-', ts.nDiasCalendario,    
                           '-', ts.nMostrar_Selector) AS sDescripcion,    
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS sTipo_Configuracion    
   from Tipos_Solicitudes ts    
                      join saldos_Vacaciones_Colaboradores svc    
                           on svc.nId_Colaborador = @Id_Colaborador    
             where ts.nEstado = 1    
               and ts.nId_Categoria IN (2)
			   ) 
			   
        END    
```

