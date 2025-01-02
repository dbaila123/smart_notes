```sql
CREATE OR ALTER PROCEDURE [dbo].[sp_Rq_Insert_Calculated_Minutes] @userId INT,
                                                         @aProjectIds NVARCHAR(MAX) = NULL
AS
BEGIN
    SET
        NOCOUNT ON;

    DECLARE @requirementId INT
    DECLARE @projectId 
INT
    DECLARE @nPendiente INT
    DECLARE @nAprobado INT
    DECLARE @nEstimado INT
    DECLARE @requirementIds TABLE
                            (
                                nId_Requerimiento INT,
                                nId_Proyecto      
INT
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

            INSERT INTO @requirementIds
      

      SELECT r.nId_Requerimiento,
                   r.nId_Proyecto
            FROM Requerimientos r
            WHERE r.nId_Proyecto IN (SELECT ProjectId FROM @projectIdsTable)
        END
    ELSE -- call from job
        BEGIN
            INSERT INTO 

@requirementIds
            SELECT nId_Requerimiento, nId_Proyecto
            FROM Requerimientos
        END

    SET @requirementId = (SELECT TOP 1 nId_Requerimiento FROM @requirementIds)
    SET @projectId = (SELECT TOP 1 nId_Proyecto FROM @requirementIds)

    WHILE @requirementId IS NOT NULL
        BEGIN
            SET
                @nPendiente = COALESCE(
                        (SELECT SUM(nMinutos)
                         FROM Tareas t
                                  JOIN Requerimientos r 

ON r.nId_Requerimiento = t.nId_Requerimiento
                         WHERE t.nId_Proyecto = @projectId
                           AND t.nId_Requerimiento = @requirementId
                           AND t.nEstado_Tarea IN (1, 2, 5, 8)),
                
        0
                              )
            SET
                @nAprobado = COALESCE(
                        (SELECT SUM(nMinutos)
                         FROM Tareas t
                        
          JOIN Requerimientos r ON r.nId_Requerimiento = t.nId_Requerimiento
                         WHERE t.nId_Proyecto = @projectId
                           AND t.nId_Requerimiento = @requirementId
                           AND t.nEstado_Tarea IN (3, 4)),
                        0
                             )
            SET
                @nEstimado = (SELECT SUM(nEstimacion)
                              FROM Requerimientos_Categoria
         
                     WHERE nId_Requerimiento = @requirementId)

            IF EXISTS (SELECT *
                       FROM pryRqMinutosCalculados
                       WHERE nId_Proyecto = @projectId AND nId_Requerimiento = @requirementId)
             

   BEGIN
                    UPDATE
                        pryRqMinutosCalculados
                    SET nPendiente       =
                            CASE
                                WHEN nPendiente <> @nPendiente THEN @nPendiente
                

                ELSE nPendiente
                                END,
                        nAprobado        =
                            CASE
                                WHEN nAprobado <> @nAprobado THEN @nAprobado
                                ELSE nAprobado
                                END,
                        nEstimado        =
                            CASE
                                WHEN nEstimado <> @nEstimado THEN @nEstimado
                                ELSE nEstimado
   



                            END,
                        nUsuario_Update  = @userId,
                        dDatetime_Update = DATEADD(HOUR, -5, GETDATE())
                    WHERE nId_Requerimiento = @requirementId
                      AND nId_Proyecto = @projectId
                END

            DELETE FROM @requirementIds WHERE nId_Requerimiento = @requirementId
            SET @requirementId = (SELECT TOP 1 nId_Requerimiento FROM @requirementIds)
            SET @projectId = (SELECT TOP 1 nId_Proyecto FROM @requirementIds)
        END
END
```
