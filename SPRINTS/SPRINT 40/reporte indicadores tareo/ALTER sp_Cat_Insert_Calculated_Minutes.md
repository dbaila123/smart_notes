```sql
CREATE OR ALTER PROCEDURE [dbo].[sp_Cat_Insert_Calculated_Minutes] @userId INT,
                                                          @aProjectIds NVARCHAR(MAX) = NULL
AS
BEGIN
    DECLARE @projectId INT
    DECLARE @requirementId INT
    DECLARE @categoryId
 INT
    DECLARE @pendingMinutes INT
    DECLARE @approvedMinutes INT
    DECLARE @estimatedMinutes INT
    DECLARE @PryRqCatIds TABLE
                         (
                             nId_Proyecto      INT,
                             nId_Requerimiento INT,
                             nId_Categoria     INT
                         )

    IF @aProjectIds IS NOT NULL -- call from refresh button
        BEGIN
            DECLARE @collaboratorId INT = (SELECT c.nId_Colaborador
                       
  
                  FROM Colaboradores c
                                                    JOIN Usuarios u ON u.nId_Persona = c.nId_Persona
                                           WHERE u.nId_Usuario = @userId)

            DECLARE @projectIdsTable TABLE
                                     (
                                         ProjectId INT
                                     )
            INSERT INTO @projectIdsTable (ProjectId)
            SELECT value
            FROM STRING_SPLIT(@aProjectIds, ',')

            INSERT INTO @PryRqCatIds
            SELECT DISTINCT r.nId_Proyecto,
                            r.nId_Requerimiento,
                            rc.nId_Categoria
            FROM Requerimientos r
                     JOIN Requerimientos_Categoria rc ON rc.nId_Requerimiento = r.nId_Requerimiento
            WHERE r.nId_Proyecto IN
                  (SELECT ProjectId
                   FROM @projectIdsTable)
        END
    ELSE -- call from job
        BEGIN
            INSERT INTO @PryRqCatIds
            SELECT p.nId_Proyecto      AS nId_Proyecto,
                   r.nId_Requerimiento AS nId_Requerimiento,
                   rc.nId_Categoria    AS nId_Categoria
            FROM Proyectos p
                     JOIN Requerimientos r

 ON p.nId_Proyecto = r.nId_Proyecto
                     JOIN Requerimientos_Categoria rc ON rc.nId_Requerimiento = r.nId_Requerimiento
        END

    SET @projectId = (SELECT TOP 1 nId_Proyecto FROM @PryRqCatIds)
    SET @requirementId = (SELECT TOP 1 nId_Requerimiento FROM @PryRqCatIds)
    SET @categoryId = (SELECT TOP 1 nId_Categoria FROM @PryRqCatIds)

    WHILE @projectId IS NOT NULL
        BEGIN
            SET @pendingMinutes = COALESCE(
                    (SELECT SUM(t_pending.nMinutos)
     

                FROM Tareas t_pending
                     WHERE t_pending.nId_Proyecto = @projectId
                       AND t_pending.nId_Requerimiento = @requirementId
                       AND CONVERT(INT, t_pending.sId_Categoria) = @categoryId
   

                    AND t_pending.nEstado_Tarea IN (1, 2, 5, 8, 9)),
                    0
                                  )

            SET @approvedMinutes = COALESCE(
                    (SELECT
 SUM(t_approved.nMinutos)
                     FROM Tareas t_approved
                     WHERE t_approved.nId_Proyecto = @projectId
                       AND t_approved.nId_Requerimiento = @requirementId
                       AND CONVERT(INT, t_approved.sId_Categoria) = @categoryId
                       AND t_approved.nEstado_Tarea IN (3, 4, 10, 11)),
                    0
                                   )

            SET @estimatedMinutes =
 (SELECT nEstimacion
                                     FROM Requerimientos_Categoria
                                     WHERE nId_Requerimiento = @requirementId AND nId_Categoria = @categoryId)

            IF EXISTS (SELECT *
                       

FROM pryRqCatMinutosCalculados
                       WHERE nId_Proyecto = @projectId
                         AND nId_Requerimiento = @requirementId
 AND nId_Categoria = @categoryId)
                BEGIN
                    UPDATE pryRqCatMinutosCalculados
            SET nPendiente       =
                            CASE
                                WHEN nPendiente <> @pendingMinutes THEN @pendingMinutes
                                ELSE nPendiente
                                END,
  
                      nAprobado        =
                            CASE
                                WHEN nAprobado <> @approvedMinutes THEN @approvedMinutes
                                ELSE nAprobado
                                END,
       
 
                nEstimado        =
                            CASE
                                WHEN nEstimado <> @estimatedMinutes THEN @estimatedMinutes
                                ELSE nEstimado
                                END,
            

            nUsuario_Update  = @userId,
                        dDatetime_Update = DATEADD(HOUR, -5, GETDATE())
                    WHERE nId_Proyecto = @projectId
                      AND nId_Requerimiento = @requirementId
                      AND nId_Categoria = @categoryId
                END

            DELETE
            FROM @PryRqCatIds
            WHERE nId_Proyecto = @projectId
              AND nId_Requerimiento = @requirementId
              AND nId_Categoria = @categoryId

            SET @projectId = (SELECT TOP 1 nId_Proyecto FROM @PryRqCatIds)
            SET @requirementId = (SELECT TOP 1 nId_Requerimiento FROM @PryRqCatIds)
            SET @categoryId = (SELECT TOP 1 nId_Categoria FROM @PryRqCatIds)
        END
END
```