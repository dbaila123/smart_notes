```sql
  
CREATE   PROCEDURE [dbo].[sp_HEstimadas_HConsumidas_dashboard_v2](  
    @Id_Colaborador INT,  
    @Fecha_Inicio DATE,  
    @Fecha_Fin DATE,  
    @sSlug NVARCHAR(MAX)  
)  
AS  
BEGIN  
    IF @sSlug IN ('TASK-DASHBOARD-TEAM', 'TASK-DASHBOARD-ALL')  
  
            SELECT p.nId_Proyecto,  
                   CONCAT(p.sNombre, ' ', '(', (SELECT TOP 1 value  
                                                from (SELECT ROW_NUMBER() OVER (ORDER BY (select null)) as row, value  
                                      FROM STRING_SPLIT(p2.sPersona_Nombre, ' ')) as nombres  
                                                order by row desc), ')') sNombre_Proyecto,  
                   r.nId_Requerimiento,  
                   r.sNombre    AS    sNombre_Requerimiento,  
                   c.nId_Categoria,  
                   c.sDescripcion             AS                         sNombre_Categoria,  
                   rc.nEstimacion,  
                   -- ISNULL(SUM(t.nMinutos), 0) AS                         nMinutos,  
       isnull((prc.nPendiente + prc.nAprobado), 0) AS nMinutos,  
       isnull(prc.nPendiente, 0) nPendiente,  
       isnull(prc.nAprobado, 0) nAprobado  
            FROM  Proyectos p  
                     JOIN Requerimientos r ON r.nId_Proyecto = p.nId_Proyecto  
  
      JOIN Requerimientos_Categoria rc ON rc.nId_Requerimiento = r.nId_Requerimiento  
                     JOIN Categorias c ON c.nId_Categoria = rc.nId_Categoria  
                     JOIN Coordinadores_Proyectos cp ON p.nId_Proyecto = cp.nId_Proyecto  
  
                  JOIN Coordinadores c2 ON cp.nId_Coordinador = c2.nId_Coordinador  
                     JOIN Personas p2 ON c2.nId_Persona = p2.nId_Persona  
      --               LEFT JOIN Tareas t ON t.nId_Proyecto = p.nId_Proyecto  
      --          AND t.nId_Requerimiento = r.nId_Requerimiento  
      --          AND t.sId_Categoria = c.nId_Categoria  
      --          AND t.dFecha_Registro BETWEEN @Fecha_Inicio AND @Fecha_Fin  
      --          AND t.nEstado = 1  
      --          AND t.sTipo_Hora NOT IN (4)  
  
      --AND t.nEstado_Tarea IN (1, 3)  
   LEFT JOIN pryRqCatMinutosCalculados prc ON prc.nId_Proyecto = p.nId_Proyecto  
                                                AND prc.nId_Requerimiento = r.nId_Requerimiento  
                                                AND prc.nId_Categoria = c.nId_Categoria  
            WHERE (@sSlug = 'TASK-DASHBOARD-TEAM' AND p.nId_Proyecto IN (SELECT nId_Proyecto  
                                                                          FROM v_Listado_Proyectos_refact  
  
                                                       WHERE nLider = @Id_Colaborador  
                                                                             OR (nDirector = @Id_Colaborador AND nPrincipal = 1)))  
               OR (@sSlug = 'TASK-DASHBOARD-ALL' AND p.nId_Proyecto IN (SELECT DISTINCT nId_Proyecto FROM Proyectos))  
      -- and p.sNombre = 'PRY - SMART' and r.sNombre = 'RQ - SPRINT 33'  
            GROUP BY p.nId_Proyecto,  
                     p.sNombre,  
                     r.nId_Requerimiento,  
                     r.sNombre,  
                     c.nId_Categoria,  
                     c.sDescripcion,  
                     rc.nEstimacion,  
                     p2.sPersona_Nombre,  
      prc.nPendiente,  
      prc.nAprobado  
            ORDER BY r.sNombre;  
        END  
```


