```sql
CREATE TABLE History_Closing_Transactions
(
    nId_Cierre INT PRIMARY KEY,
    dDatetime_Creacion DATETIME,
    nUsuario_Creador INT,
    dCantidad_Minutos INT,
    Json_Detalles_Transacciones NVARCHAR(MAX),
);
```