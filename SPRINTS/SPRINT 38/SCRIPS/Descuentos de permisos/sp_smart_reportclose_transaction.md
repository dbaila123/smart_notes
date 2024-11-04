```sql
CREATE   PROCEDURE sp_smart_reportclose_transaction
	@nId_Cierre INT
AS 
BEGIN
DECLARE @nId_Colaborador INT;

    SELECT @nId_Colaborador = nId_Colaborador
    FROM Transacciones_Saldo_Mins
    WHERE nId_Cierre = @nId_Cierre;

    IF @nId_Colaborador IS NOT NULL
    BEGIN
        SELECT * 
        FROM v_listado_Transacciones_Saldo_
Mins
        WHERE nId_Colaborador = @nId_Colaborador;
   END
END;

```