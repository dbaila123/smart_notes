```SQL
-- Inserción en Type_Question
INSERT INTO Type_Question (sName, nUser_Creator, dDatetime_Creator)
VALUES 
    ('opcion mutiple', 172, GETDATE()),
    ('opcion unica', 172, GETDATE()),
    ('texto libre', 172, GETDATE()),
    ('seleccionar lista', 172, GETDATE());

-- Inserción en Form_Template
INSERT INTO Form_Template (sName, sDescription, dDatetime_Creator)
VALUES 
    ('Evaluación 270', 'Para pares de líderes',  GETDATE()),
    ('Evaluación 270', 'Para pares de subordinados', GETDATE());

-- Inserción en Questions_Template
INSERT INTO Questions_Template (nId_Form_Template, sText, nId_Type_Question, bRequired, nId_Competence, nUser_Creator, dDatetime_Creator)
VALUES 
    (1, 'Comunica información de manera clara y precisa, asegurándose de que sea relevante, comprensible y promueva la transparencia', 2, 1, 1, 172, GETDATE()),
    (1, 'Participa activamente en las actividades del equipo y colabora con otros miembros del equipo y/o áreas para fomentar sinergias', 2, 1, 2, 172, GETDATE()),
    (1, 'Recopila información clave de diversas fuentes para resolver problemas de forma efectiva, mostrando iniciativa', 2, 1, 3, 172, GETDATE()),
    (1, 'Demuestra flexibilidad y determinación a nuevos procesos y tareas, adaptándose de forma efectiva a los cambios', 2, 1, 4, 172, GETDATE()),
    (1, 'Planifica y organiza sus tareas diarias para cumplir con las metas propuestas de manera eficiente y ágil', 2, 1, 5, 172, GETDATE()),
    (1, 'Construye relaciones de confianza y respeto mutuo con sus colegas, promoviendo un ambiente colaborativo', 2, 1, 2, 172, GETDATE());