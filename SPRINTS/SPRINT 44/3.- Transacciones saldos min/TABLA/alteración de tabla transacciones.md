```sql
ALTER TABLE Transacciones_Saldo_Mins
ADD nEstado_Entidad INT NULL;

UPDATE Transacciones_Saldo_Mins
SET nEstado_Entidad = 3
WHERE nEstado_Entidad IS NULL;
```
