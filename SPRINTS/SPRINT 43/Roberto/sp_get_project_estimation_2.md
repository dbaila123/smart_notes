```sql

CREATE PROCEDURE [dbo].[sp_get_project_estimation_2]
(
    @nId_Lider INT,        -- ID del Líder (ahora siempre es requerido)
    @nEstado_Proyecto INT = NULL,  -- Estado del Proyecto (puede ser NULL para traer todos)
    @sSlug NVARCHAR(MAX)   -- Tipo de vista (TEAM o ALL)
)
AS
BEGIN
    SET NOCOUNT ON;


    -- Determinar la lista de proyectos según el tipo de vista (sSlug)
    WITH CTE_Proyectos AS (
        SELECT nId_Proyecto 
        FROM v_Listado_Proyectos_refact 
        WHERE nLider = @nId_Lider OR nDirector = @nId_Lider -- TEAM

        UNION ALL


        SELECT DISTINCT nId_Proyecto 
        FROM Proyectos 
        WHERE @sSlug = 'TASK-DASHBOARD-ALL'  -- ALL
    )
    
    -- Consulta principal con los proyectos filtrados
    SELECT	
        pr.sEstado_Proyecto AS nId_Estado,
        (SELECT sDescripcion 
         FROM Configs 
         WHERE sCodigo = CONVERT(NVARCHAR, pr.sEstado_Proyecto) 
           AND sTabla = 'ESTADO_PROYECTO') AS sEstado,
        pr.nId_Lider,
        (SELECT Per.sPersona_Nombre 
         FROM Colaboradores C 
         INNER JOIN Personas Per ON C.nId_Persona = Per.nId_Persona
         WHERE C.nId_Colaborador = pr.nId_Lider) AS sNombre_Lider,
        pr.nId_Proyecto,
        pr.sNombre AS sNombre_Proyecto, 
        SUM(pry.nPendiente) AS Min_Pendientes,
        SUM(pry.nAprobado) AS Min_Aprobadas, 
        SUM(pry.nEstimado) AS Min_Estimados 
    FROM pryRqCatMinutosCalculados pry
    LEFT JOIN Proyectos pr ON pry.nId_Proyecto = pr.nId_Proyecto 
    WHERE pr.nId_Proyecto IN (SELECT nId_Proyecto FROM CTE_Proyectos)  -- Usar la CTE para filtrar proyectos
      AND (pr.sEstado_Proyecto = ISNULL(@nEstado_Proyecto, pr.sEstado_Proyecto)) -- Filtro por estado del proyecto
    GROUP BY pr.sEstado_Proyecto, pr.nId_Lider, pr.sNombre, pr.nId_Proyecto
    ORDER BY pr.sNombre;
END;




```