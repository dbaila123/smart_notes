```sql
create procedure sp_update_transaction_Request_subtype
AS
BEGIN
	 SET NOCOUNT ON;

    DECLARE @dFecha_actual DATE = GETDATE();

    UPDATE t
    SET t.nId_Sub_Tipo_Entidad = 6
    FROM Transacciones_Saldo_Mins t
    JOIN Solicitudes s ON t.nId_Entidad = s.nId_Solictud
    WHERE t.nId_Sub_Tipo_Entidad = 7
    	AND nId_Cierre IS NULL
    	 AND DATEDIFF(DAY, s.dFecha_Inicio, @dFecha_actual) > 30;


    SELECT @@ROWCOUNT AS RowsAffected;
END


```