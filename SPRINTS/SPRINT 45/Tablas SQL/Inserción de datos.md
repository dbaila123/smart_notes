```sql
INSERT INTO Form_Type (sName, nUser_Creator, dDateTime_Creator, nId_FormType_Padre)
VALUES ('Colaborador-Lideres', 666, GETDATE(), NULL),
('Lideres-Equipos', 666, GETDATE(), NULL),
('Pares', 666, GETDATE(), NULL),
('Pares-Lideres', 666, GETDATE(), NULL),
('Pares-Colaborador', 666, GETDATE(), NULL)

INSERT INTO Competence (sName, sDescription, sCode, nUser_Creator, dDateTime_Creator)
VALUES ('Comunicación', 'Competencia Comunicación', 'C', 666, GETDATE()),
('Trabajo en equipo', 'Competencia Trabajo en equipo', 'TE', 666, GETDATE()),
('Resolución de problemas', 'Competencia Resolución de problemas', 'RP', 666, GETDATE()),
('Mejora continua', 'Competencia Mejora continua', 'MC', 666, GETDATE()),
('Organización y administración del tiempo', 'Competencia Organización y administración del tiempo', 'OT', 666, GETDATE()),
('Enfoque en el cliente', 'Competencia Enfoque en el cliente', 'EC', 666, GETDATE()),
('Pensamiento estratégico', 'Competencia Pensamiento estratégico', 'PE', 666, GETDATE()),
('Enfoque a resultados', 'Competencia Enfoque a resultados', 'ER', 666, GETDATE()),
('Liderazgo', 'Competencia Liderazgo', 'L', 666, GETDATE()),
('Calidad de trabajo', 'Competencia Calidad de trabajo', 'CT', 666, GETDATE())


```