```SQL
ALTER TABLE Cargos 
ADD nId_Job_Category INT;

ALTER TABLE Cargos 
ADD CONSTRAINT FK_Cargos_Job_Category
FOREIGN KEY (nId_Job_Category) REFERENCES Job_Category(nId_Job_Category);

UPDATE Cargos
SET nId_Job_Category = 1
WHERE nId_Cargo in (1,6,39,41);

UPDATE Cargos
SET nId_Job_Category = 3
WHERE nId_Cargo in (7,10,12,13,14,15,32,34,38,40,43,45,46,47,48,49);

UPDATE Cargos
SET nId_Job_Category = 4
WHERE nId_Cargo in (3,37);

UPDATE Cargos
SET nId_Job_Category = 5
WHERE nId_Cargo in (17,31,42);