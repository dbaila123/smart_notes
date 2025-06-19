```SQL
ALTER TABLE Cargos 
ADD nId_Job_Category INT;

ALTER TABLE Cargos 
ADD CONSTRAINT FK_Cargos_Job_Category
FOREIGN KEY (nId_Job_Category) REFERENCES Job_Category(nId_Job_Category);

UPDATE Cargos
SET nId_Job_Category = 1
WHERE nId_Cargo in (39,41);

UPDATE Cargos
SET nId_Job_Category = 2
WHERE nId_Cargo in(6,53,59);

UPDATE Cargos
SET nId_Job_Category = 3
WHERE nId_Cargo in (10,13,14,15,32,34,40,43,45,46,47,48,49,51,52,54,55,57,58);

UPDATE Cargos
SET nId_Job_Category = 4
WHERE nId_Cargo in (3,37,50,56);

UPDATE Cargos
SET nId_Job_Category = 5
WHERE nId_Cargo in (17,31,42);

UPDATE Cargos
SET nId_Job_Category = 6
WHERE nId_Cargo = 1
