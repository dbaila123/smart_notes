```sql
CREATE OR ALTER PROCEDURE [dbo].[sp_smart_get_proy_notification_hours_exceeded]    
(    
 @nId_Lider int    
)    
AS    
BEGIN    
 SELECT    
  P.nId_Proyecto,    
  UPPER(concat(p.sCodigo,' ', p.sNombre_Proyecto_Sin_Prefijo)) as sProyecto,    
  R.nId_Requerimiento,    
  UPPER(CONCAT(r.sCodigo, ' ', r.sNombre_SinPrefijo)) AS sRequerimiento,
  PRCN.nNotificacion_Status nNotification_Status,    
  PryReqCat.nPendiente,    
  PryReqCat.nAprobado,    
  PryReqCat.nEstimado FROM Proyectos P    
 JOIN Requerimientos R ON P.nId_Proyecto = R.nId_Proyecto    
 JOIN (SELECT TP.nId_Requerimiento, SUM(TP.nEstimado) nEstimado, SUM(TP.nAprobado) nAprobado, SUM(TP.nPendiente) nPendiente    
 FROM pryRqCatMinutosCalculados TP GROUP BY TP.nId_Requerimiento) PryReqCat ON R.nId_Requerimiento = PryReqCat.nId_Requerimiento    
 JOIN Proyecto_Requerimiento_Colaborador_Notificacion PRCN ON PRCN.nId_Requerimiento = R.nId_Requerimiento    
 WHERE (YEAR(R.dDatetime_Creador) >= 2024 AND MONTH(R.dDatetime_Creador) > 1 AND R.nEstado = 1)    
 AND PRCN.nEstado = 1 AND PRCN.nId_Colaborador = @nId_Lider AND PRCN.nNotificacion_Status = 1    
 AND (PryReqCat.nPendiente + PryReqCat.nAprobado) > PryReqCat.nEstimado    
END;
```

