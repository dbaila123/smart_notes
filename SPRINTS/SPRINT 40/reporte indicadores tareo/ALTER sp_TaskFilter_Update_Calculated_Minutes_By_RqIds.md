```sql
CREATE OR ALTER PROCEDURE [dbo].[sp_TaskFilter_Update_Calculated_Minutes_By_RqIds]  
 @userId INT,  
 @requirementIds NVARCHAR(MAX) = NULL  
AS  
BEGIN  
 DECLARE @isSuccess BIT = 0  
 BEGIN TRY  
 BEGIN TRANSACTION  
 DECLARE @requirementIdsTable TABLE (Id INT)  
  
 IF @requirementIds IS NOT NULL -- UPDATE FILTERS (WEB)  
 BEGIN  
 INSERT INTO @requirementIdsTable (Id)  
 SELECT value  
 FROM STRING_SPLIT(@requirementIds, ',')  
 END  
 ELSE -- UPDATE massively (JOB)  
 BEGIN  
 INSERT INTO @requirementIdsTable (Id)  
 SELECT nId_Requerimiento  
 FROM Requerimientos  
 END  
  
 UPDATE pryRqMinutosCalculados  
 SET  
 nPendiente = (  
 SELECT COALESCE(SUM(nMinutos), 0)  
 FROM Tareas  
 WHERE nEstado_Tarea IN (1, 2, 5, 8, 9) AND  
 nId_Requerimiento = pryRqMinutosCalculados.nId_Requerimiento  
 ),  
 nAprobado = (  
 SELECT COALESCE(SUM(nMinutos), 0)  
 FROM Tareas  
 WHERE nEstado_Tarea IN (3, 4, 10, 11) AND  
 nId_Requerimiento = pryRqMinutosCalculados.nId_Requerimiento  
 ),  
 nEstimado = (  
 SELECT COALESCE(SUM(rc.nEstimacion), 0)  
 FROM Requerimientos_Categoria rc  
 JOIN Categorias c ON c.nId_Categoria = rc.nId_Categoria  
 WHERE nId_Requerimiento = pryRqMinutosCalculados.nId_Requerimiento  
 ),  
 dFecha_Inicio_Pendiente = (  
 select min(t.dFecha_Registro) from Tareas t where t.nId_Proyecto = pryRqMinutosCalculados.nId_Proyecto and t.nId_Requerimiento = pryRqMinutosCalculados.nId_Requerimiento and t.nEstado_Tarea = 1  
 ),  
 dFecha_Fin_Pendiente = (  
 select max(t.dFecha_Registro) from Tareas t where t.nId_Proyecto = pryRqMinutosCalculados.nId_Proyecto and t.nId_Requerimiento = pryRqMinutosCalculados.nId_Requerimiento and t.nEstado_Tarea = 1  
 ),  
 nUsuario_Update = @userId,  
 dDatetime_Update = DATEADD(HOUR, -5, GETDATE())  
 WHERE nId_Requerimiento IN (SELECT Id FROM @requirementIdsTable)  
  
 UPDATE pryRqCatMinutosCalculados  
 SET  
 nPendiente = (  
 SELECT COALESCE(SUM(nMinutos), 0)  
 FROM Tareas  
 WHERE nEstado_Tarea IN (1, 2, 5, 8, 9) AND  
 nId_Requerimiento = pryRqCatMinutosCalculados.nId_Requerimiento AND  
 sId_Categoria = CAST(pryRqCatMinutosCalculados.nId_Categoria AS NVARCHAR(50))  
 ),  
 nAprobado = (  
 SELECT COALESCE(SUM(nMinutos), 0)  
 FROM Tareas  
 WHERE nEstado_Tarea IN (3, 4, 10, 11) AND  
 nId_Requerimiento = pryRqCatMinutosCalculados.nId_Requerimiento AND  
 sId_Categoria = CAST(pryRqCatMinutosCalculados.nId_Categoria AS NVARCHAR(50))  
 ),  
 nEstimado = (  
 SELECT nEstimacion  
 FROM Requerimientos_Categoria rc  
 JOIN Categorias c ON c.nId_Categoria = rc.nId_Categoria  
 WHERE nId_Requerimiento = pryRqCatMinutosCalculados.nId_Requerimiento AND  
 rc.nId_Categoria = pryRqCatMinutosCalculados.nId_Categoria  
 ),  
 dFecha_Inicio_Pendiente = (  
 select min(t.dFecha_Registro) from Tareas t where t.nId_Requerimiento = pryRqCatMinutosCalculados.nId_Requerimiento and t.sId_Categoria = convert(varchar, pryRqCatMinutosCalculados.nId_Categoria) and t.nEstado_Tarea = 1  
 ),  
 dFecha_Fin_Pendiente = (  
 select max(t.dFecha_Registro) from Tareas t where t.nId_Requerimiento = pryRqCatMinutosCalculados.nId_Requerimiento and t.sId_Categoria = convert(varchar, pryRqCatMinutosCalculados.nId_Categoria) and t.nEstado_Tarea = 1  
 ),  
 nUsuario_Update = @userId,  
 dDatetime_Update = DATEADD(HOUR, -5, GETDATE())  
 WHERE nId_Requerimiento IN (SELECT Id FROM @requirementIdsTable)  
 COMMIT TRANSACTION  
 SET @isSuccess = 1  
 END TRY  
 BEGIN CATCH  
 ROLLBACK TRANSACTION  
 SET @isSuccess = 0  
 END CATCH  
END
```
