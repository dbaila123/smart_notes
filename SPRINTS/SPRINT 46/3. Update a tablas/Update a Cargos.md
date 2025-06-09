```SQL
ALTER TABLE Cargos 
ADD nId_Hierarchy_Level INT;

ALTER TABLE Cargos 
ADD CONSTRAINT FK_Cargos_Hierarchy_Level
FOREIGN KEY (nId_Hierarchy_Level) REFERENCES Hierarchy_Level(nId_Hierarchy_Level);

UPDATE Cargos
SET nId_Hierarchy_Level = 1
WHERE nId_Cargo in (1,6,39,41);

UPDATE Cargos
SET nId_Hierarchy_Level = 3
WHERE nId_Cargo in (7,10,12,13,14,15,32,34,38,40,43,45,46,47,48,49);

UPDATE Cargos
SET nId_Hierarchy_Level = 4
WHERE nId_Cargo in (3,37);

UPDATE Cargos
SET nId_Hierarchy_Level = 5
WHERE nId_Cargo in (17,31,42);