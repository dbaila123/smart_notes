```SQL
CREATE FUNCTION fn_nEntidad_Transaction_By_Request_Date(

@dFecha_Inicio DATE, -- FECHA DE INICIO SOLICITUD

@dFecha_Fin DATE -- FECHA DE COMPARACIÓN

)

RETURNS INT

AS

BEGIN

DECLARE @nDias_Cierre INT = (SELECT dbo.fn_get_closing_days_by_request_date(@dFecha_Inicio))--(SELECT CAST(sCodigo AS INT) FROM Configs WHERE sTabla = 'VENCIMIENTO-SOLICITUDES');

DECLARE @diff INT;

DECLARE @nEntidadVencida INT = (SELECT nId_Entidad FROM Entidades_Transacciones WHERE sDescripcion = 'SOLICITUDES VENCIDAS');

DECLARE @nEntidaSinVencer INT = (SELECT nId_Entidad FROM Entidades_Transacciones WHERE sDescripcion = 'SOLICITUDES SIN VENCER');

  

SET @diff = DATEDIFF(DAY, @dFecha_Inicio, @dFecha_Fin);

  

RETURN CASE

WHEN @diff >= @nDias_Cierre THEN @nEntidadVencida

WHEN @diff < @nDias_Cierre THEN @nEntidaSinVencer

ELSE NULL

END;

END
