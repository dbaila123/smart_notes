``` sql
USE [tenant0001]
GO
/****** Object:  UserDefinedFunction [dbo].[fn_get_request_hours_by_Transaction_All]    Script Date: 22/04/2025 15:52:49 ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER   FUNCTION [dbo].[fn_get_request_hours_by_Transaction_All](  
 @sId_Collaborator NVARCHAR(MAX) = NULL,        
    @bOnlyPendingRequest BIT = 1,        
    @bShowNegativeDiff BIT = 0,        
    @bOnlyRequestEnding BIT = 1,        
    @nId_Cierre INT = NULL,        
    @dFecha_Corte        
        
 DATETIME = NULL        
)        
RETURNS TABLE        
AS        
RETURN        
    SELECT  
 tsm.nId_Colaborador,  
 SUM(CASE  
            WHEN @bOnlyRequestEnding = 1 AND tsm.nId_Sub_Tipo_Entidad = 6 AND tsm.nEstado_Entidad IN (1,3,4) THEN tsm.dCantidad_Minutos  
            WHEN @bOnlyRequestEnding = 0 AND tsm.nId_Sub_Tipo_Entidad IN(6, 7) AND tsm.nEstado_Entidad IN (1,3,4) THEN tsm.dCantidad_Minutos  
            ELSE 0  
        END) AS nPermisosAP,  
    SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 AND tsm.nEstado_Entidad IN (1,3,4) AND tsm.nId_Tipo_Entidad != 5 THEN tsm.dCantidad_Minutos ELSE 0 END) as nAFavorAP,  
    SUM(CASE  
            WHEN @bOnlyRequestEnding = 1 AND tsm.nId_Sub_Tipo_Entidad = 6 AND tsm.nEstado_Entidad = 3 THEN tsm.dCantidad_Minutos  
            WHEN @bOnlyRequestEnding = 0 AND tsm.nId_Sub_Tipo_Entidad IN(6, 7) AND tsm.nEstado_Entidad = 3 THEN tsm.dCantidad_Minutos  
            ELSE 0  
        END) AS nPermisosApro, 
    SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 AND tsm.nEstado_Entidad = 3 THEN tsm.dCantidad_Minutos ELSE 0 END) as nAFavorApro,  
    SUM(CASE  
            WHEN @bOnlyRequestEnding = 1 AND tsm.nId_Sub_Tipo_Entidad = 6 AND tsm.nEstado_Entidad IN (1,4) THEN tsm.dCantidad_Minutos  
            WHEN @bOnlyRequestEnding = 0 AND tsm.nId_Sub_Tipo_Entidad IN(6, 7) AND tsm.nEstado_Entidad IN (1,4) THEN tsm.dCantidad_Minutos  
            ELSE 0  
        END) AS nPermisosPen,  
    SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 8 AND tsm.nEstado_Entidad IN (1,4)  THEN tsm.dCantidad_Minutos ELSE 0 END) as nAFavorPen,  
 SUM(CASE WHEN tsm.nId_Tipo_Entidad = 5 AND tsm.nId_Sub_Tipo_Entidad = 5 THEN tsm.dCantidad_Minutos ELSE 0 END) as nBolsa  ,
    SUM(CASE WHEN tsm.nId_Tipo_Entidad=9 and tsm.nId_Sub_Tipo_Entidad = 8 AND tsm.nEstado_Entidad = 3  THEN tsm.dCantidad_Minutos ELSE 0 END) as nTotal_Favor_CompeRecu_A

FROM Transacciones_Saldo_Mins tsm   
JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador  
WHERE ((@nId_Cierre IS NULL AND tsm.nId_Cierre IS NULL) OR (@nId_Cierre IS NOT NULL AND tsm.nId_Cierre = @nId_Cierre))  
    AND tsm.nId_Tipo_Entidad IN (1, 9, 5)  
    AND (@sId_Collaborator IS NULL OR tsm.nId_Colaborador IN (SELECT value FROM STRING_SPLIT(@sId_Collaborator, ',')))  
    AND (@dFecha_Corte IS NULL OR (@dFecha_Corte IS NOT NULL AND CONVERT(DATE, tsm.dDatetime_Creacion) <= CONVERT(DATE,@dFecha_Corte)))  
    AND c.nEstado_Colaborador = 1  
GROUP BY  
    tsm.nId_Colaborador  
HAVING  
    0 < (CASE  
          WHEN @bOnlyPendingRequest = 1 THEN  
             SUM(CASE WHEN tsm.nId_Sub_Tipo_Entidad = 6 THEN tsm.dCantidad_Minutos ELSE 0 END)  
          ELSE 1  
         END)
```