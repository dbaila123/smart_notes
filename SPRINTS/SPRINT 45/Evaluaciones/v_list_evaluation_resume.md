```SQL
CREATE OR ALTER VIEW v_list_evaluation_resume
AS
	WITH Evaluators_Completed as (
		SELECT 
		        e.nId_Evaluator,
		        f.nId_Evaluation,
		        f.nId_Form,
		        CASE
		            WHEN COUNT(*) = COUNT(CASE WHEN qe.nId_Qualified_Evaluation IS NOT NULL THEN 1 END) 
		            THEN 1
		            ELSE 0
		        END
		        as Answered
		    FROM Evaluators e
		    JOIN Forms f on e.nId_Form = f.nId_Form
		    LEFT JOIN Qualified_Evaluations qe on e.nId_Evaluator = qe.nId_Evaluator AND e.nId_Assessed = qe.nId_Assessed AND e.nId_Form = qe.nId_Form 
		    GROUP BY 
		        e.nId_Evaluator, 
		        f.nId_Evaluation, 
		        f.nId_Form
		
		), ExpectedEvaluatorQuantities AS (
		    
		    SELECT
		        f.nId_Evaluation,
		        COUNT(DISTINCT e.nId_Evaluator) as Expected
		    FROM Evaluators e
		    JOIN Forms f on e.nId_Form = f.nId_Form
		    GROUP BY f.nId_Evaluation
		
		), EvaluationsResults AS (
		    SELECT 
		        eeq.nId_Evaluation,
		        eeq.Expected,
		        COUNT(CASE WHEN ec.Answered = 1 THEN 1 END) AS Answered
		    FROM ExpectedEvaluatorQuantities eeq
		    JOIN (
		        select 
		            e.nId_Evaluator,
		            e.nId_Evaluation,
		            CASE 
		                WHEN MIN(e.Answered) = 1 THEN 1
		                ELSE 0
		            END AS Answered
		        from Evaluators_Completed e
		        group by e.nId_Evaluator,
		            e.nId_Evaluation
		    ) ec on eeq.nId_Evaluation = ec.nId_Evaluation
		    GROUP BY eeq.nId_Evaluation, eeq.Expected
		) 
		SELECT
		    e.nId_Evaluation,
		    e.sName AS sEvaluation_Name,
			e.sDescription AS sEvaluation_Description,
		    e.dInit_Date AS dStart_Date,
		    e.dEnd_Date AS dEnd_Date,
			e.dDateTime_Creator,
			CASE 
				WHEN e.nState = 1 THEN 1
				WHEN GETDATE() BETWEEN e.dInit_Date AND e.dEnd_Date THEN 2
				WHEN GETDATE() < e.dInit_Date THEN 4
				WHEN GETDATE() > e.dEnd_Date THEN 3
			END AS nState,
		    p.sPersona_Nombre AS sCreator,
		    CAST(er.Answered AS VARCHAR) + '/' + CAST(er.Expected AS VARCHAR) AS Participants,
			ROUND(CAST(er.Answered AS FLOAT) / er.Expected, 4) AS Progress
		FROM Evaluations e
		JOIN Usuarios u ON e.nUser_Creator = u.nId_Usuario
		JOIN Personas p ON u.nId_Persona = p.nId_Persona
		JOIN Forms f ON e.nId_Evaluation = f.nId_Evaluation
		JOIN Evaluators eva ON f.nId_Form = eva.nId_Form
		JOIN EvaluationsResults er ON e.nId_Evaluation = er.nId_Evaluation
		GROUP BY 
			e.nId_Evaluation, 
			e.sName,
			e.dInit_Date, 
			e.dEnd_Date, 
			p.sPersona_Nombre,
			er.Answered, 
			er.Expected, 
			e.sDescription, 
			e.nState,
			e.dDateTime_Creator
```