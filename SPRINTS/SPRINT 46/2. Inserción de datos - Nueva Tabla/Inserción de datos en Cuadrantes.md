```sql
INSERT INTO Evaluation_Quadrant(sName, sDescription, nCompetence_Min_Value, nCompetence_Max_Value, nObjetive_Min_Value, nObjetive_Max_Value, nUser_Creator, dDateTime_Creator)
VALUES ('Enigma', 'Investigar motivos de su bajo desempeño. Necesita entrenamiento y ayuda del profesional comprometido', 3.33, 5, 0, 1.66, 172, GETDATE()),
('Dilema', 'Ofrecer entrenamiento para mejorar su rol actual', 1.67, 3.32, 0, 1.66, 172, GETDATE()),
('Bajo desempeño', 'Reconsiderar su futuro', 0, 1.66, 0, 1.66, 172, GETDATE()),
('Estrella a futuro', 'Talento valioso. Desarrollable en el puesto actual', 3.33, 5, 1.67, 3.32, 172, GETDATE()),
('Clave', 'Con potencial para seguir creciendo en su rol actual', 1.67, 3.32, 1.67, 3.32, 172, GETDATE()),
('Eficaz', 'Hacerte un acompañamiento para su mejora personal', 0, 1.66, 1.67, 3.32, 172, GETDATE()),
('Súper estrella', 'Listo para promocionar y afrontar nuevos desafios', 3.33, 5, 3.33, 5, 172, GETDATE()),
('Estrella en su área', 'Contribuye de manera valiosa. Motivar para que apunte más alto',1.67, 3.32, 3.33, 5, 172, GETDATE()),
('Comprometido', 'Muy valioso pero no promocionable. Puede ayudar al desarrollo de otros',0, 1.66, 3.33, 5, 172, GETDATE())
```