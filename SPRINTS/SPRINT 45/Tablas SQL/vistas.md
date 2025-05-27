```sql
CREATE OR ALTER VIEW list_evaluation_questions 
AS
	SELECT 
		f.nId_Evaluation,
	    f.nId_Form,
		CASE 
			WHEN f.nId_Form_SubType IS NULL THEN f.nId_Form_Type
			ELSE f.nId_Form_SubType
		END AS nId_Form_Type,
		ft.sName as Form_Type_Name,
		q.nId_Question,
	    q.sText,
	    c.nId_Competence,
	    c.sName as Competence_Name,
	    tq.nId_Type_Question,
	    tq.sName as sQuestion_Type_Name,
	    q.bRequired
	FROM Forms f 
	JOIN Questions q on f.nId_Form = q.nId_Form
	JOIN Type_Question tq on q.nId_Type_Question = tq.nId_Type_Question
	JOIN Competence c on q.nId_Competence = c.nId_Competence
	JOIN Form_Type ft on f.nId_Form_Type = ft.nId_Form_Type;
GO

CREATE OR ALTER VIEW v_list_evaluators_assessed
AS
	SELECT 
		e2.nId_Evaluator,
		f2.nId_Evaluation,
		c2.nId_Colaborador AS nId_Assessed,
		p2.sPersona_Nombre AS sAssessed_Name,
		ft2.nId_Form_Type,
		ft2.sName AS sFormType_Name,
		ft2.sIcon,
		CASE 
			WHEN qe.nId_Qualified_Evaluation IS NOT NULL THEN 1
			ELSE 2
		END AS nState
	FROM Evaluators e2
	JOIN Colaboradores c2 ON e2.nId_Assessed = c2.nId_Colaborador
	JOIN Personas p2 ON c2.nId_Persona = p2.nId_Persona
	JOIN Forms f2 ON e2.nId_Form = f2.nId_Form
	JOIN Form_Type ft2 ON ft2.nId_Form_Type = f2.nId_Form_Type 
	LEFT JOIN Qualified_Evaluations qe ON e2.nId_Evaluator = qe.nId_Evaluator AND e2.nId_Assessed = qe.nId_Assessed AND f2.nId_Form = qe.nId_Form;
GO

CREATE OR ALTER VIEW v_list_evaluators_form_completition_status
AS
	SELECT 
	    e.nId_Evaluator,
		p.sPersona_Nombre AS sEvaluator_Name,
	    f.nId_Evaluation,
		f.nId_Form_Type,
	    CASE
	        WHEN COUNT(*) = COUNT(CASE WHEN qe.nId_Qualified_Evaluation IS NOT NULL THEN 1 END) 
	        THEN 1
	        ELSE 0
	    END
	    AS nAnswered
	FROM Evaluators e
	JOIN Colaboradores c on e.nId_Evaluator = c.nId_Colaborador
	JOIN Personas p on c.nId_Persona = p.nId_Persona
	JOIN Forms f on e.nId_Form = f.nId_Form
	LEFT JOIN Qualified_Evaluations qe on e.nId_Evaluator = qe.nId_Evaluator AND e.nId_Assessed = qe.nId_Assessed AND e.nId_Form = qe.nId_Form 
	GROUP BY 
	    e.nId_Evaluator, 
	    f.nId_Evaluation,
		f.nId_Form_Type,
		p.sPersona_Nombre;
GO

CREATE OR ALTER VIEW list_evaluation_questions_options
AS 
	SELECT 
		*,
		(
	        SELECT
	            dl.nId_Data_List,
	            dl.sText,
	            dl.nValue
	        FROM data_list dl
	        WHERE dl.nId_Question = leq.nId_Question
	        FOR JSON PATH
	    ) as aOptions
	FROM list_evaluation_questions leq;
GO

CREATE OR ALTER VIEW v_list_evaluator_assessed_form_questions
AS 
	SELECT 
		leq.nId_Evaluation,
		e.nId_Evaluator,
		e.nId_Assessed,
		e.nId_Form,
		leq.nId_Question,
		leq.sText,
		leq.nId_Competence,
		leq.Competence_Name AS sCompetence_Name,
		leq.nId_Type_Question,
		leq.bRequired,
		(
			SELECT
			    dl.nId_Data_List,
			    dl.sText,
			    dl.nValue,
				CASE
					WHEN dl.nId_Data_List = dla.nId_Data_List THEN 1 
					ELSE 0 
				END AS bSelected
			FROM data_list dl
			LEFT JOIN Data_List_Answer dla ON dl.nId_Data_List= dla.nId_Data_List AND dla.nId_Answer = a.nId_Answer
			WHERE dl.nId_Question = leq.nId_Question
			FOR JSON PATH
		) as aOptions,
		qe.dDateTime_Creator AS dDate_Answered,
		a.sText_Answer,
		a.nNumber_Answer
	FROM Evaluators e
	JOIN list_evaluation_questions leq ON e.nId_Form = leq.nId_Form
	LEFT JOIN Qualified_Evaluations qe ON e.nId_Evaluator = qe.nId_Evaluator AND e.nId_Assessed = qe.nId_Assessed AND e.nId_Form = qe.nId_Form 
	LEFT JOIN Answer a ON qe.nId_Qualified_Evaluation = a.nId_Qualified_Evaluation and leq.nId_Question = a.nId_Question;
GO

CREATE OR ALTER VIEW v_list_evaluation_resume
AS
	WITH EvaluationsResults AS (
		    SELECT 
		        ec.nId_Evaluation,
		        COUNT(CASE WHEN ec.nAnswered = 1 THEN 1 END) + COUNT(CASE WHEN ec.nAnswered = 0 THEN 1 END) AS Expected,
		        COUNT(CASE WHEN ec.nAnswered = 1 THEN 1 END) AS nAnswered
		    FROM (
		        select 
		            e.nId_Evaluator,
		            e.nId_Evaluation,
		            MIN(e.nAnswered) AS nAnswered
		        from v_list_evaluators_form_completition_status e
		        group by e.nId_Evaluator,
		            e.nId_Evaluation
		    ) ec
		    GROUP BY ec.nId_Evaluation
		) 
		SELECT
		    e.nId_Evaluation,
		    e.sName AS sEvaluation_Name,
			e.sDescription AS sEvaluation_Description,
		    e.dInit_Date AS dStart_Date,
		    e.dEnd_Date AS dEnd_Date,
			e.dDateTime_Creator,
			(
				SELECT 
					* 
				FROM (
					SELECT DISTINCT ft.sIcon, ft.sName
					FROM Form_Type ft
					JOIN Forms f1 on ft.nId_Form_Type = f1.nId_Form_Type
					WHERE f1.nId_Evaluation = e.nId_Evaluation) as s
				FOR JSON PATH
			) AS sIcons,
			CASE 
				WHEN e.nState = 1 THEN 1
				WHEN GETDATE() BETWEEN e.dInit_Date AND e.dEnd_Date THEN 2
				WHEN GETDATE() < e.dInit_Date THEN 4
				WHEN GETDATE() > e.dEnd_Date THEN 3
			END AS nState,
		    p.sPersona_Nombre AS sCreator,
		    CAST(er.nAnswered AS VARCHAR) + '/' + CAST(er.Expected AS VARCHAR) AS sParticipants,
			ROUND(CAST(er.nAnswered AS FLOAT) / er.Expected, 4) * 100 AS nProgress
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
			er.nAnswered, 
			er.Expected, 
			e.sDescription, 
			e.nState,
			e.dDateTime_Creator;
GO

CREATE OR ALTER VIEW v_list_evaluation_create_resume
AS 
	SELECT	
		e.nId_Evaluation,
		e.sName,
		e.sDescription,
		e.dInit_Date,
		e.dEnd_Date,
		(
			SELECT 
				ft.nId_Form_Type,
				ft.sName,
				ft.sIcon,
				CASE 
					WHEN MAX(f.nId_Form_SubType) IS NOT NULL THEN NULL
					ELSE MAX(f.nId_Form_Template)
				END AS nId_Template,
				CASE 
					WHEN MAX(f.nId_Form_SubType) IS NOT NULL THEN NULL
					ELSE MAX(ftem.sName)
				END AS sTemplate_Name,
				CASE 
					WHEN MAX(f.nId_Form_SubType) IS NOT NULL THEN NULL
					ELSE (select COUNT(DISTINCT e.nId_Evaluator) from Evaluators e where e.nId_Form = MAX(f.nId_Form))
				END AS nParticipants,
				(
					SELECT
						ft2.nId_Form_Type,
						ft2.sName,
						ft2.sIcon,
						f2.nId_Form_Template,
						ftem2.sName as sTemplate_Name,
						(select COUNT(DISTINCT e.nId_Evaluator) from Evaluators e where e.nId_Form =f2.nId_Form) AS nParticipants
					FROM Forms f2
					JOIN Form_Type ft2 ON ft2.nId_Form_Type = f2.nId_Form_SubType
					JOIN Form_Template ftem2 ON f2.nId_Form_Template = ftem2.nId_Form_Template
					WHERE f2.nId_Evaluation = f.nId_Evaluation
					  AND f2.nId_Form_Type = f.nId_Form_Type
					FOR JSON PATH
				) AS sSubtypes
			FROM Forms f
			JOIN Form_Type ft ON ft.nId_Form_Type = f.nId_Form_Type
			JOIN Form_Template ftem ON f.nId_Form_Template = ftem.nId_Form_Template
			WHERE f.nId_Evaluation = e.nId_Evaluation
			GROUP BY ft.nId_Form_Type, ft.sName, ft.sIcon, f.nId_Form_Type, f.nId_Evaluation
			FOR JSON PATH
		) AS sTypes
	FROM Evaluations e;
GO
```