```sql
CREATE VIEW v_list_evaluators_assessed

AS

SELECT

c.nId_Colaborador AS nId_Evaluator,

p.sPersona_Nombre AS sName_Evaluator,

f.nId_Evaluation,

(

SELECT

c2.nId_Colaborador AS nId_Assessed,

p2.sPersona_Nombre AS sAssessed_Name,

ft2.nId_Form_Type,

ft2.sName AS sFormType_Name

FROM Evaluators e2

JOIN Colaboradores c2 ON e2.nId_Assessed = c2.nId_Colaborador

JOIN Personas p2 ON c2.nId_Persona = p2.nId_Persona

JOIN Forms f2 ON e2.nId_Form = f2.nId_Form

JOIN Form_Type ft2 ON ft2.nId_Form_Type = f2.nId_Form_Type

WHERE e2.nId_Evaluator = e.nId_Evaluator and f2.nId_Evaluation = f.nId_Evaluation

ORDER BY ft2.nId_Form_Type

FOR JSON PATH

) AS sEvaluators

FROM Evaluators e

JOIN Colaboradores c ON e.nId_Evaluator = c.nId_Colaborador

JOIN Personas p ON c.nId_Persona = p.nId_Persona

JOIN Forms f on e.nId_Form = f.nId_Form

GROUP BY C.nId_Colaborador, p.sPersona_Nombre, e.nId_Evaluator, f.nId_Evaluation;
```