```SQL

-- COLABORADORES
ALTER TABLE Colaboradores
ADD dfec_ingreso_real DATETIME;

UPDATE Colaboradores
SET dfec_ingreso_real = dFec_Ingreso;

```