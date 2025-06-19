```SQL
-- Tablas que dependen de otras
DROP TABLE IF EXISTS Competence_Average_Evaluator;
DROP TABLE IF EXISTS Competence_Average;
DROP TABLE IF EXISTS Job_Hierarchy_Weight;
DROP TABLE IF EXISTS Job_Competency_Weight;
DROP TABLE IF EXISTS Objective_Score;

-- Tablas base que no dependen de otras 
DROP TABLE IF EXISTS Evaluation_Quadrant;
DROP TABLE IF EXISTS Hierarchy_Level;
DROP TABLE IF EXISTS Job_Category;