```SQL 
CREATE OR ALTER PROCEDURE [dbo].[sp_smart_evaluation_create](
	@UserCreator INT,
	@sTitle NVARCHAR(MAX),
	@sDescription NVARCHAR(MAX),
	@dInit_Date DATETIME,
	@dEnd_Date DATETIME,
	@sForms NVARCHAR(MAX)
)
AS 
BEGIN
	--SET NOCOUNT ON;
	BEGIN TRY
        BEGIN TRANSACTION;
			DECLARE @FormTypeTemplate AS TABLE (nId_FormType INT, nId_Template INT);
			
			INSERT INTO @FormTypeTemplate
			SELECT 
				f.nId_FormType,
				f.nId_Template
			FROM OPENJSON(@sForms)
			WITH (
				nId_FormType INT,
				nId_Template INT
			) as f

			IF EXISTS( 
				SELECT 1
				FROM @FormTypeTemplate ftt 
				LEFT JOIN Form_Template ft on ftt.nId_Template = ft.nId_Form_Template
				WHERE ft.nId_Form_Template IS NULL
			)
			BEGIN
				 RAISERROR('No existe una de las plantillas', 16, 1);
			END

			IF EXISTS( 
				SELECT 1
				FROM @FormTypeTemplate ftt 
				LEFT JOIN Form_Type ft on ftt.nId_FormType = ft.nId_Form_Type
				WHERE ft.nId_Form_Type IS NULL OR ft.nId_Form_Type IN (
						SELECT 
							form.nId_FormType_Padre
						FROM Form_Type form 
						WHERE form.nId_FormType_Padre IS NOT NULL
					)
			)
			BEGIN
				 RAISERROR('No existe uno de los tipos de evaluaciones', 16, 1);
			END


			--INSERT EVALUATION
			DECLARE @nId_Evaluation INT;
			INSERT INTO Evaluations (sName, nState, sDescription, dInit_Date, dEnd_Date, nUser_Creator, dDateTime_Creator)
			VALUES (@sTitle, 1, @sDescription, @dInit_Date, @dEnd_Date, @UserCreator, GETDATE());

			SET @nId_Evaluation = SCOPE_IDENTITY();

			--INSERT FORMS 
			DECLARE @FormTable AS TABLE (nId_Form INT ,nId_Form_Type INT, nId_Form_SubType INT);
			INSERT INTO Forms (sName, sDescription, nId_Evaluation, nUser_Creator, dDateTime_Creator, nId_Form_Type, nId_Form_SubType, nId_Form_Template)
			OUTPUT inserted.nId_Form, inserted.nId_Form_Type, inserted.nId_Form_SubType INTO @FormTable
			SELECT 
				ft.sName,
				ft.sName as sDescription,
				@nId_Evaluation as nId_Evaluation,
				@UserCreator as nUser_Creator,
				GETDATE() as dDateTime_Creator,
				(CASE 
					WHEN ft.nId_FormType_Padre IS NULL THEN ft.nId_Form_Type 
					ELSE ft.nId_FormType_Padre 
				END) as nId_Form_Type,
				(CASE 
					WHEN ft.nId_FormType_Padre IS NULL THEN NULL
					ELSE ft.nId_Form_type 
				END) as nId_Form_SubType,
				t.nId_Template as nId_Form_Template
			FROM @FormTypeTemplate t
			JOIN Form_Type ft on t.nId_FormType = ft.nId_Form_Type

			--INSERT QUESTIONS
			CREATE TABLE #QuestionsMap (
			    nId_Question_Template INT,
			    nId_Question INT
			);

			INSERT INTO Questions (nId_Form, sText, nId_Type_Question, bRequired, sPlaceHolder, nId_Competence, nUser_Creator, dDateTime_Creator, nId_Question_Template)
			OUTPUT inserted.nId_Question_Template, inserted.nId_Question INTO #QuestionsMap
			SELECT 
				formt.nId_Form,
				qt.sText,
				qt.nId_Type_Question,
				qt.bRequired,
				qt.sPlaceHolder,
				qt.nId_Competence,
				@UserCreator,
				GETDATE() as dDateTime_Creator,
				qt.nId_Question_Template
			FROM Form_Template ft
			JOIN Questions_Template qt ON ft.nId_Form_Template = qt.nId_Form_Template
			JOIN @FormTypeTemplate ftt ON ft.nId_Form_Template = ftt.nId_Template
			JOIN @FormTable formt ON (
			 (formt.nId_Form_SubType IS NULL AND ftt.nId_FormType = formt.nId_Form_Type)
			 OR
			 (formt.nId_Form_SubType IS NOT NULL AND ftt.nId_FormType = formt.nId_Form_SubType)
			) ORDER BY formt.nId_Form
			
			--INSERT data lists from templates
			INSERT INTO data_list (
			    nId_Question,
			    sText,
			    nValue,
			    nUser_Creator,
			    dDatetime_Creator
			)
			SELECT 
			    qm.nId_Question,
			    dlt.sText,
			    dlt.nValue,
			    @UserCreator,
			    GETDATE()
			FROM dataList_Template dlt
			JOIN #QuestionsMap qm ON dlt.nId_Question = qm.nId_Question_Template;

			--INSERT EVALUATORS

			DECLARE @FormTypes NVARCHAR(100);
			SELECT 
				@FormTypes = STRING_AGG(t.nId_FormType, ',')
			FROM @FormTypeTemplate t

			CREATE TABLE #Evaluators (
				nId_Evaluator INT,
				sEvaluator_Name VARCHAR(MAX),
				nId_Assessed INT,
				sAssessed_Name VARCHAR(MAX),
				nId_FormType INT
			)

			INSERT INTO #Evaluators 
			EXEC sp_smart_evaluation_get_evaluators_assessed_template NULL, @FormTypes;

			INSERT INTO Evaluators
			SELECT 
				e.nId_Evaluator,
				e.nId_Assessed,
				ft.nId_Form
			FROM #Evaluators e
			JOIN @FormTable ft ON 
			(
			 (ft.nId_Form_SubType IS NULL AND e.nId_FormType = ft.nId_Form_Type)
			 OR
			 (ft.nId_Form_SubType IS NOT NULL AND e.nId_FormType = ft.nId_Form_SubType)
			)

			SELECT 
				@nId_Evaluation AS nId_Evaluation

		COMMIT TRANSACTION;

    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        THROW;
    END CATCH

END;
GO

CREATE OR ALTER PROCEDURE [dbo].[sp_smart_evaluation_edit_evaluators_assessed](
	@nId_Evaluation NVARCHAR(MAX),
	@nId_Evaluator INT,
	@AssessedList AssessedFormTypeList READONLY
)
AS 
BEGIN
	BEGIN TRY
        BEGIN TRANSACTION;
			
			--Comprobar si es supervisor
			DECLARE @IsSupervisor BIT;

			SELECT @IsSupervisor = 
					CASE
						WHEN EXISTS (
								SELECT 1
								FROM @AssessedList al
								WHERE al.nId_FormType = 3 --tipo subordinados
							) THEN 1
						ELSE 0
					END

			--BULK DELETE
			DELETE e
			FROM Evaluators e
			JOIN Forms f ON e.nId_Form = f.nId_Form
			WHERE e.nId_Evaluator = @nId_Evaluator AND f.nId_Evaluation = @nId_Evaluation

			--INSERT si el formulario no tiene subtipos
			
			INSERT INTO Evaluators
			SELECT 
				@nId_Evaluator AS nId_Evaluator,
				al.nId_Assessed,
				f.nId_Form
			FROM @AssessedList al
			JOIN Forms f ON al.nId_FormType = f.nId_Form_Type
			WHERE f.nId_Form_SubType IS NULL AND f.nId_Evaluation = @nId_Evaluation

			--INSERTS EN CASO DE PARES
			IF EXISTS (
					SELECT 1 FROM @AssessedList WHERE nId_FormType = 6
				)
			BEGIN

				INSERT INTO Evaluators
				SELECT 
					@nId_Evaluator AS nId_Evaluator,
					al.nId_Assessed,
					f.nId_Form
				FROM @AssessedList al
				JOIN Forms f ON al.nId_FormType = f.nId_Form_Type
				WHERE f.nId_Form_SubType IS NOT NULL 
					AND f.nId_Evaluation = @nId_Evaluation 
					AND f.nId_Form_Type = 6 --tipo pares
					AND f.nId_Form_SubType = 
						(CASE
							WHEN @IsSupervisor = 1 THEN 8 --Lideres
							ELSE 7 --Colaborador
						END)
			END

			--Caso Subordinados

			IF EXISTS (
					SELECT 1 FROM @AssessedList WHERE nId_FormType = 3
				)
			BEGIN

				DECLARE @nNewId_Form INT

				--Obtener evaluado y a que subtipo de evaluacion pertenece
				SELECT 
					al.nId_Assessed,
					CASE 
						WHEN f.nId_Form_Type IS NULL THEN 4
						ELSE 5
					END as nId_Form_SubType
				INTO #Subordinates_supervisor
				FROM @AssessedList al
				LEFT JOIN Evaluators e ON al.nId_Assessed = e.nId_Evaluator 
					AND e.nId_Form IN (
							SELECT
								form.nId_Form
							FROM forms form
							WHERE form.nId_Evaluation = @nId_Evaluation
							AND form.nId_Form_Type = 3
						)
				LEFT JOIN Forms f on e.nId_Form = f.nId_Form
				WHERE al.nId_FormType = 3
				GROUP BY al.nId_Assessed, f.nId_Form_Type

				--Insert de evaluados Subordinados
				INSERT INTO Evaluators
				SELECT 
					@nId_Evaluator AS nId_Evaluator,
					al.nId_Assessed,
					f.nId_Form
				FROM #Subordinates_supervisor al
				JOIN Forms f ON al.nId_Form_SubType = f.nId_Form_SubType
				WHERE f.nId_Evaluation = @nId_Evaluation

			END
			--update del evaluador donde es evaluado 
			SELECT 
				f.nId_Form,
				f.nId_Form_SubType
			INTO #Forms
			FROM Forms f
			WHERE f.nId_Evaluation = 21
			AND f.nId_Form_Type = 3

			SELECT @nNewId_Form = f2.nId_Form
			FROM #Forms f2
			WHERE f2.nId_Form_SubType = 
				CASE 
				    WHEN @IsSupervisor = 1 THEN 5
				    ELSE 4
				END;

			UPDATE e
			SET e.nId_Form = @nNewId_Form
			FROM Evaluators e
			JOIN Forms f ON e.nId_Form = f.nId_Form
			WHERE 
				e.nId_Assessed = @nId_Evaluator 
				AND f.nId_Evaluation = @nId_Evaluation
				AND f.nId_Form_Type = 3

		COMMIT TRANSACTION;
    END TRY
	BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        THROW;
    END CATCH
END;
GO

CREATE OR ALTER PROCEDURE sp_smart_evaluation_evaluator_assessed_form_questions
(
	@nId_Evaluation INT,
	@nId_Evaluator VARCHAR(MAX) = NULL,
	@nId_Assessed VARCHAR(MAX) = NULL,
	@pnum INT = NULL, --Número de Páginas
    @size INT = NULL --Cantidad de Registros por página
)
AS 
BEGIN
	DECLARE @script AS nvarchar(max)
	DECLARE @whereClause AS nvarchar(max) = ''
	DECLARE @paginationClause AS nvarchar(max)
    DECLARE @conditions AS TABLE (condition nvarchar(max))
	DECLARE @conditionOUT AS nvarchar(max);
	
	IF @nId_Evaluator IS NOT NULL
	BEGIN
		INSERT INTO @conditions VALUES (N'e.nId_Evaluator IN (' + @nId_Evaluator + ')');
	END

	IF @nId_Assessed IS NOT NULL
	BEGIN
		INSERT INTO @conditions VALUES (N'e.nId_Assessed IN (' + @nId_Assessed + ')');
	END

	--ADD CONDITIONS
	DECLARE @count INT

	SELECT @count = COUNT(*) FROM @conditions
	
	SET @whereClause = ''

	WHILE @count > 0
	BEGIN
		DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)

		SET @whereClause = concat (@whereClause,'and ', @condition_temp)
			
		DELETE TOP (1) FROM @conditions
		SELECT @count = COUNT(*) FROM @conditions
	END


	--PAGINATION
	IF @pnum IS NULL AND @size IS NULL
    BEGIN
        SET @paginationClause = ''
    END
    ELSE
    BEGIN
    	EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT
    END
	SET @script = N'
		SELECT *
		FROM v_list_evaluator_assessed_form_questions e
		WHERE e.nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation) + @whereClause +'
		ORDER BY e.nId_Question
		' + @paginationClause
	EXECUTE sp_executesql @script
END;
GO

CREATE OR ALTER PROCEDURE sp_smart_evaluation_getFormTypeAndTemplate
AS
BEGIN
    SET NOCOUNT ON;
    
    SELECT 
        nId_Form_Type AS nId,
        sName,
        sIcon,
        sDescription
    FROM Form_Type
    WHERE nId_FormType_Padre IS NULL;
    
    -- Obtener los subtipos con su padre
    SELECT 
        ft.nId_Form_Type AS nId,
        ft.sName,
        ft.nId_FormType_Padre AS nIdParent
    FROM Form_Type ft
    WHERE ft.nId_FormType_Padre IS NOT NULL;
    
    /*SELECT 
        nId_Form_Template AS nId,
        sName,
        sDescription
    FROM Form_Template;*/
END;
GO

CREATE OR ALTER PROCEDURE sp_smart_evaluation_get_create_resume
(
	@nId_Evaluation INT
)
AS
BEGIN
	SELECT *
	FROM v_list_evaluation_create_resume l
	WHERE l.nId_Evaluation = @nId_Evaluation
END;
GO

CREATE OR ALTER PROCEDURE sp_smart_evaluation_get_evaluation_resume
(
	@sEvaluation_Name VARCHAR(MAX) = NULL,
	@dStartDate VARCHAR(MAX) = NULL,
	@dEndDate VARCHAR(MAX) = NULL, 
	@sFilterOne VARCHAR(MAX) = NULL, --FILTRO POR ESTADO
	@pnum INT = NULL, --Número de Páginas
    @size INT = NULL --Cantidad de Registros por página
)
AS 
BEGIN
	DECLARE @script AS nvarchar(max)
	DECLARE @whereClause AS nvarchar(max) = ''
	DECLARE @paginationClause AS nvarchar(max)
    DECLARE @conditions AS TABLE (condition nvarchar(max))
	DECLARE @conditionOUT AS nvarchar(max);
	
	IF @sEvaluation_Name IS NOT NULL
	BEGIN 
		EXEC sp_get_like_condition 'sEvaluation_Name', @sEvaluation_Name, @conditionOUT OUTPUT
        INSERT INTO @conditions VALUES (@conditionOUT);
	END

	IF @sFilterOne IS NOT NULL
	BEGIN
		INSERT INTO @conditions VALUES (N'nState IN (' + @sFilterOne + ')');
	END

	IF @dStartDate IS NOT NULL AND @dEndDate IS NOT NULL
    BEGIN
		SET @dStartDate = CONCAT(@dStartDate, ' 00:00:00.000')
		SET @dEndDate = CONCAT(@dEndDate, ' 23:59:59.999')

		SELECT @conditionOUT = CONCAT('dDateTime_Creator',' BETWEEN ''', CONVERT(DATETIME2(3), @dStartDate),''' AND ''', CONVERT(DATETIME2(3), @dEndDate), '''');
        INSERT INTO @conditions VALUES (@conditionOUT)
    END
	--Agregar condicionales
	DECLARE @count INT

	SELECT @count = COUNT(*) FROM @conditions
	IF @count > 0
		BEGIN
			SET @whereClause = N'WHERE '
		END

	WHILE @count > 0
		BEGIN
			DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)
			IF @count = 1
				BEGIN
					SET @whereClause = concat (@whereClause,@condition_temp)
				END
			ELSE
				BEGIN
					SET @whereClause = concat (@whereClause,@condition_temp,' and ')
				END
			DELETE TOP (1) FROM @conditions
			SELECT @count = COUNT(*) FROM @conditions
		END

	--PAGINATION
	IF @pnum IS NULL AND @size IS NULL
    BEGIN
        SET @paginationClause = ''
    END
    ELSE
    BEGIN
    	EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT
    END

	--Principal Query
	SET @script = N'
		SELECT 
			*,
			COUNT(*) OVER() AS nTotal_Registers
		FROM v_list_evaluation_resume
		'
		+ @whereClause + 
		'
		ORDER BY nId_Evaluation ' + @paginationClause

	EXECUTE sp_executesql @script

END;
GO

CREATE OR ALTER PROCEDURE sp_smart_evaluation_get_evaluators_assessed
(
	@nId_Evaluation INT,
	@sFilterOne VARCHAR(MAX) = NULL,--FILTRO EQUIPOS
	@sFilterTwo VARCHAR(MAX) = NULL,-- FILTRO POR COLABORADOR
	@sFilterThree VARCHAR(MAX) = NULL, --FILTRO POR ESTADO DE CUESTIONARIO (RESPONDIDO, POR RESPONDER)
	@pnum INT = NULL, --Número de Páginas
    @size INT = NULL, --Cantidad de Registros por página
	@sEvaluator_Name VARCHAR(MAX) = NULL
)
AS 
BEGIN
	DECLARE @script AS nvarchar(max)
	DECLARE @whereClause AS nvarchar(max)
	DECLARE @paginationClause AS nvarchar(max)
    DECLARE @conditions AS TABLE (condition nvarchar(max))
	DECLARE @conditionOUT AS nvarchar(max);
	DECLARE @whereClauseState AS NVARCHAR(MAX) = '';
	--TEAM FILTER
	IF @sFilterOne IS NOT NULL
  	BEGIN
   		DECLARE @CollaboratorsListTeam VARCHAR(MAX);
   		SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)
   		INSERT INTO @conditions VALUES(N'e.nId_Evaluator in('+@CollaboratorsListTeam+')')
  	END
	
	IF @sFilterTwo IS NOT NULL
  	BEGIN
   		INSERT INTO @conditions VALUES(N'e.nId_Evaluator = '+@sFilterTwo)
  	END

	IF @sEvaluator_Name IS NOT NULL
	BEGIN 
		EXEC sp_get_like_condition 'p.sName_Evaluator', @sEvaluator_Name, @conditionOUT OUTPUT
        INSERT INTO @conditions VALUES (@conditionOUT);
	END

	IF @sFilterThree IS NOT NULL
	BEGIN
		SET @whereClauseState = 'AND lea.nState IN('+@sFilterThree+')'
	END

	--ADD CONDITIONS
	DECLARE @count INT

	SELECT @count = COUNT(*) FROM @conditions
	
	SET @whereClause = ''

	WHILE @count > 0
	BEGIN
		DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)

		SET @whereClause = concat (@whereClause,'and ', @condition_temp)
			
		DELETE TOP (1) FROM @conditions
		SELECT @count = COUNT(*) FROM @conditions
	END

	--PAGINATION
	IF @pnum IS NULL AND @size IS NULL
    BEGIN
        SET @paginationClause = ''
    END
    ELSE
    BEGIN
    	EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT
    END

	--Principal query
	SET @script = N'
		SELECT
			c.nId_Colaborador AS nId_Evaluator,
			p.sPersona_Nombre AS sName_Evaluator,
			f.nId_Evaluation, 
			e1.sName as sEvaluation_Name,
			e1.sDescription,
			e1.dInit_Date,
			e1.dEnd_Date,
			CASE 
				WHEN e1.nState = 1 THEN 1
				WHEN GETDATE() BETWEEN e1.dInit_Date AND e1.dEnd_Date THEN 2
				WHEN GETDATE() < e1.dInit_Date THEN 4
				WHEN GETDATE() > e1.dEnd_Date THEN 3
			END AS nState,
			(
				SELECT 
					lea.nId_Assessed,
					lea.sAssessed_Name,
					lea.nId_Form_Type,
					lea.sFormType_Name,
					lea.sIcon,
					lea.nState
				FROM v_list_evaluators_assessed lea
				WHERE lea.nId_Evaluator = e.nId_Evaluator AND lea.nId_Evaluation = f.nId_Evaluation '+@whereClauseState+'
				ORDER BY lea.nId_Form_Type
				FOR JSON PATH
			) AS sEvaluators,
			COUNT(*) OVER() AS nTotal_Registers
		FROM Evaluators e
		JOIN Colaboradores c ON e.nId_Evaluator = c.nId_Colaborador
		JOIN Personas p ON c.nId_Persona = p.nId_Persona
		JOIN Forms f on e.nId_Form = f.nId_Form
		JOIN Evaluations e1 ON f.nId_Evaluation = e1.nId_Evaluation
		WHERE f.nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation) + @whereClause +'
		GROUP BY c.nId_Colaborador, p.sPersona_Nombre, e.nId_Evaluator, f.nId_Evaluation, e1.sName, e1.dInit_Date, e1.dEnd_Date, e1.sDescription, e1.nState
		ORDER BY e.nId_Evaluator ' +
		@paginationClause

	EXECUTE sp_executesql @script

END;
GO

CREATE OR ALTER PROCEDURE sp_smart_evaluation_get_evaluators_assessed_template(
@nId_Collaborator NVARCHAR(MAX) = NULL,
@nTipo_Evaluation NVARCHAR(MAX) --IDs de la tabla Form_Type
)
AS 
BEGIN
	DECLARE @aColaboradores TABLE (nId_Colaborador INT);
	
	INSERT INTO @aColaboradores (nId_Colaborador)
    SELECT CAST(value AS INT) AS nId_Colaborador
    FROM STRING_SPLIT(@nId_Collaborator, ',');

	DECLARE @aFormType TABLE (nId_FormType INT);
	INSERT INTO @aFormType (nId_FormType)
    SELECT CAST(value AS INT) AS nId_FormType
    FROM STRING_SPLIT(@nTipo_Evaluation, ',');

	CREATE TABLE #Evaluators (
        nId_Evaluator INT,
        sEvaluator_Name NVARCHAR(100),
        nId_Assessed INT,
        sAssessed_Name NVARCHAR(100),
		nId_FormType INT
    );

	--Create tabla lideres a equipos
	SELECT 
		c.nId_Colaborador as nId_Evaluator,
		pc.sPersona_Nombre as Evaluator_Name,
		cs.nId_Colaborador as nId_Assessed,
		pcs.sPersona_Nombre as Assessed_Name
	INTO #Lideres_Equipos
	FROM Colaboradores c
	JOIN Personas pc on c.nId_Persona = pc.nId_Persona
	JOIN colaboradores_supervisor cs 
		ON c.nId_Colaborador = cs.nId_Supervisor 
		AND cs.nEstado = 1 
		AND cs.bPrincipal = 1
	JOIN Colaboradores ccs on cs.nId_Colaborador = ccs.nId_Colaborador 
	JOIN Personas pcs on ccs.nId_Persona = pcs.nId_Persona
	WHERE c.nEstado_Colaborador = 1
		  AND ccs.nEstado_Colaborador = 1

	---------Supervisores
   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 2)
	  OR NOT EXISTS(SELECT 1 FROM @aFormType) 
	BEGIN
	    INSERT INTO #Evaluators (nId_Evaluator, sEvaluator_Name, nId_Assessed, sAssessed_Name, nId_FormType)
	    SELECT 
	        c.nId_Colaborador AS nId_Evaluator,
	        p1.sPersona_Nombre AS Evaluator_Name,
	        cs.nId_Supervisor AS nId_Assessed,
	        p2.sPersona_Nombre AS Assessed_Name,
			2
	    FROM colaboradores_supervisor cs
	    INNER JOIN Colaboradores c ON cs.nId_Colaborador = c.nId_Colaborador
	    INNER JOIN Personas p1 ON c.nId_Persona = p1.nId_Persona
	    INNER JOIN Colaboradores s ON cs.nId_Supervisor = s.nId_Colaborador
	    INNER JOIN Personas p2 ON s.nId_Persona = p2.nId_Persona
	    WHERE (
			    NOT EXISTS (SELECT 1 FROM @aColaboradores)
				OR c.nId_Colaborador IN (SELECT nId_Colaborador FROM @aColaboradores)
			  ) 
	    AND cs.nEstado = 1
	    AND c.nEstado_Colaborador = 1
		AND cs.bPrincipal = 1
	    AND s.nEstado_Colaborador = 1;
	END

	------------Subordinados Colaborador
   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 4) 
	  OR NOT EXISTS(SELECT 1 FROM @aFormType) 
   BEGIN
		INSERT INTO #Evaluators
		SELECT 
			le.nId_Evaluator,
			le.Evaluator_Name,
			le.nId_Assessed,
			le.Assessed_Name,
			4
		FROM #Lideres_Equipos le
		WHERE le.nId_Assessed NOT IN (
				SELECT 
					l.nId_Evaluator
				FROM #Lideres_Equipos l
			)
   END

   --------------Subordinados Lideres
   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 5) 
	  OR NOT EXISTS(SELECT 1 FROM @aFormType) 
   BEGIN
		INSERT INTO #Evaluators
		SELECT 
			le.nId_Evaluator,
			le.Evaluator_Name,
			le.nId_Assessed,
			le.Assessed_Name,
			5
		FROM #Lideres_Equipos le
		WHERE 
			(
			    NOT EXISTS (SELECT 1 FROM @aColaboradores)
				OR le.nId_Evaluator IN (SELECT nId_Colaborador FROM @aColaboradores)
			) 
			AND le.nId_Assessed IN (
				SELECT 
					l.nId_Evaluator
				FROM #Lideres_Equipos l
			)
   END

   --Pares Colaboradores 

   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 7)
	  OR NOT EXISTS(SELECT 1 FROM @aFormType) 
   BEGIN
		SELECT 
	        c.nId_Colaborador AS nId_Evaluator,
	        p1.sPersona_Nombre AS Evaluator_Name,
	        cs.nId_Supervisor AS nId_Assessed,
	        p2.sPersona_Nombre AS Assessed_Name
		INTO #Colaborador_Lideres
	    FROM colaboradores_supervisor cs
	    INNER JOIN Colaboradores c ON cs.nId_Colaborador = c.nId_Colaborador
	    INNER JOIN Personas p1 ON c.nId_Persona = p1.nId_Persona
	    INNER JOIN Colaboradores s ON cs.nId_Supervisor = s.nId_Colaborador
	    INNER JOIN Personas p2 ON s.nId_Persona = p2.nId_Persona
	    WHERE (
			    NOT EXISTS (SELECT 1 FROM @aColaboradores)
				OR c.nId_Colaborador IN (SELECT nId_Colaborador FROM @aColaboradores)
			  ) 
	    AND cs.nEstado = 1
	    AND c.nEstado_Colaborador = 1
		AND cs.bPrincipal = 1
	    AND s.nEstado_Colaborador = 1;
		------------------
		INSERT INTO #Evaluators
		SELECT 
			cl.nId_Evaluator,
			cl.Evaluator_Name,
			le.nId_Assessed,
			le.Assessed_Name,
			7 as nId_FormType
		FROM #Colaborador_Lideres cl
		JOIN #Lideres_Equipos le ON cl.nId_Assessed = le.nId_Evaluator
		WHERE 
			cl.nId_Evaluator != le.nId_Assessed
			and cl.nId_Evaluator not in (select nId_Evaluator
			from #Lideres_Equipos)

   END

   ---------Pares Lideres
   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 8)
	  OR NOT EXISTS(SELECT 1 FROM @aFormType) 
	BEGIN
	    DECLARE @EsLider BIT = 0;
	    
	    IF @nId_Collaborator IS NOT NULL
	    BEGIN
	        SELECT @EsLider = 1
	        FROM colaboradores_supervisor cs
	        WHERE cs.nId_Supervisor = @nId_Collaborator
	        AND cs.nEstado = 1;
	    END
	    
	    INSERT INTO #Evaluators
	    SELECT 
	        Evaluador.nId_Colaborador AS nId_Evaluator,
	        p1.sPersona_Nombre AS sEvaluator_Name,
	        Evaluado.nId_Colaborador AS nId_Assessed,
	        p2.sPersona_Nombre AS sAssessed_Name,
			8 as nId_FormType
	    FROM 
	        (SELECT DISTINCT cs.nId_Supervisor AS nId_Colaborador
	         FROM colaboradores_supervisor cs 
	         WHERE cs.nId_Supervisor IS NOT NULL
	         AND cs.nEstado = 1
	         AND EXISTS (
	             SELECT 1 
	             FROM colaboradores_supervisor cs2 
	             WHERE cs2.nId_Colaborador = cs.nId_Supervisor 
	             AND cs2.nId_Supervisor IS NOT NULL
	         )
	         AND (@nId_Collaborator IS NULL OR @EsLider = 0 OR cs.nId_Supervisor = @nId_Collaborator)) Evaluador
	    CROSS JOIN 
	        (SELECT DISTINCT cs.nId_Supervisor AS nId_Colaborador
	         FROM colaboradores_supervisor cs 
	         WHERE cs.nId_Supervisor IS NOT NULL
	         AND cs.nEstado = 1
	         AND EXISTS (
	             SELECT 1 
	             FROM colaboradores_supervisor cs2 
	             WHERE cs2.nId_Colaborador = cs.nId_Supervisor 
	             AND cs2.nId_Supervisor IS NOT NULL
	         )) Evaluado
	    INNER JOIN Colaboradores c1 ON Evaluador.nId_Colaborador = c1.nId_Colaborador
	    INNER JOIN Personas p1 ON c1.nId_Persona = p1.nId_Persona
	    INNER JOIN Colaboradores c2 ON Evaluado.nId_Colaborador = c2.nId_Colaborador
	    INNER JOIN Personas p2 ON c2.nId_Persona = p2.nId_Persona
	    -- Excluir auto-evaluación
	    WHERE Evaluador.nId_Colaborador <> Evaluado.nId_Colaborador
	    AND c1.nEstado_Colaborador = 1
	    AND c2.nEstado_Colaborador = 1;
	END

	

   SELECT *
   FROM #Evaluators
   ORDER BY nId_Evaluator, nId_FormType
END;
GO

CREATE OR ALTER PROCEDURE sp_smart_evaluation_get_forms_questions
(
	@nId_Evaluation INT,
	@nId_Form_Type INT
)
AS
BEGIN
	SELECT
		*
	FROM list_evaluation_questions_options leq
	WHERE 
		leq.nId_Evaluation = @nId_Evaluation AND
		leq.nId_Form_Type = @nId_Form_Type
END;
GO

CREATE OR ALTER PROCEDURE sp_smart_evaluation_get_forms_resume
(
	@nId_Evaluation INT,
	@sEvaluator_Name VARCHAR(MAX) = NULL,
	@sFilterOne VARCHAR(MAX) = NULL --FILTRO POR EQUIPO

)
AS 
BEGIN
	DECLARE @script AS nvarchar(max)
	DECLARE @whereClause AS nvarchar(max) = ''
    DECLARE @conditions AS TABLE (condition nvarchar(max))
	DECLARE @conditionOUT AS nvarchar(max);

	IF @sEvaluator_Name IS NOT NULL
	BEGIN 
		EXEC sp_get_like_condition 'e.sEvaluator_Name', @sEvaluator_Name, @conditionOUT OUTPUT
        INSERT INTO @conditions VALUES (@conditionOUT);
	END

	IF @sFilterOne IS NOT NULL
  	BEGIN
   		DECLARE @CollaboratorsListTeam VARCHAR(MAX);
   		SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)
   		INSERT INTO @conditions VALUES(N'e.nId_Evaluator in('+@CollaboratorsListTeam+')')
  	END

	--ADD CONDITIONS
	DECLARE @count INT

	SELECT @count = COUNT(*) FROM @conditions
	
	SET @whereClause = ' '

	WHILE @count > 0
	BEGIN
		DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)

		SET @whereClause = concat (@whereClause,' and ', @condition_temp)
			
		DELETE TOP (1) FROM @conditions
		SELECT @count = COUNT(*) FROM @conditions
	END

	SET @script= N'
		SELECT 
		    ec.nId_Evaluation,
		    COUNT(CASE WHEN ec.nAnswered = 0 THEN 1 END) AS nPending,
		    COUNT(CASE WHEN ec.nAnswered = 1 THEN 1 END) AS nAnswered,
			(
				SELECT 
					ft.nId_Form_Type,
					ft.sName,
					ft.sIcon,
					COUNT(CASE WHEN e.nAnswered = 1 THEN 1 END) AS nAnswered,
					COUNT(CASE WHEN e.nAnswered = 0 THEN 1 END) AS nPending,
					COUNT(CASE WHEN e.nAnswered = 1 THEN 1 END) +
					COUNT(CASE WHEN e.nAnswered = 0 THEN 1 END) AS nTotal
				FROM v_list_evaluators_form_completition_status e
				JOIN Form_Type ft on e.nId_Form_Type = ft.nId_Form_Type
				WHERE e.nId_Evaluation = ec.nId_Evaluation ' + @whereClause+ '
				GROUP BY 
					ft.nId_Form_Type,
					ft.sName,
					ft.sIcon
				FOR JSON PATH
			) as sResume
		FROM (
		    select 
		        e.nId_Evaluator,
		        e.nId_Evaluation,
		        MIN(e.nAnswered) AS nAnswered
		    from v_list_evaluators_form_completition_status e
			WHERE e.nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation) + @whereClause +' 
		    group by e.nId_Evaluator,
		        e.nId_Evaluation
		) ec
		GROUP BY ec.nId_Evaluation
	'

	EXECUTE sp_executesql @script

END;
GO

CREATE OR ALTER PROCEDURE sp_smart_evaluation_get_formtype_details
(
	@nId_Evaluation INT,
	@nId_Form_Type INT,
	@sEvaluator_Name VARCHAR(MAX) = NULL,
	@sFilterOne VARCHAR(MAX) = NULL,--FILTRO POR EQUIPO
	@sFilterTwo VARCHAR(MAX) = NULL, --FILTRO POR ESTADO
	@pnum INT = NULL, --Número de Páginas
    @size INT = NULL --Cantidad de Registros por página
)
AS 
BEGIN 
	DECLARE @script AS nvarchar(max)
	DECLARE @whereClause AS nvarchar(max) = ''
	DECLARE @paginationClause AS nvarchar(max)
    DECLARE @conditions AS TABLE (condition nvarchar(max))
	DECLARE @conditionOUT AS nvarchar(max);

	IF @sEvaluator_Name IS NOT NULL
	BEGIN 
		EXEC sp_get_like_condition 'e.sEvaluator_Name', @sEvaluator_Name, @conditionOUT OUTPUT
        INSERT INTO @conditions VALUES (@conditionOUT);
	END

	IF @sFilterOne IS NOT NULL
  	BEGIN
   		DECLARE @CollaboratorsListTeam VARCHAR(MAX);
   		SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)
   		INSERT INTO @conditions VALUES(N'e.nId_Evaluator in('+@CollaboratorsListTeam+')')
  	END

	IF @sFilterTwo IS NOT NULL
	BEGIN
   		INSERT INTO @conditions VALUES(N'e.nAnswered in('+@sFilterTwo+')')
	END

	--ADD CONDITIONS
	DECLARE @count INT

	SELECT @count = COUNT(*) FROM @conditions
	
	SET @whereClause = ' '

	WHILE @count > 0
	BEGIN
		DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)

		SET @whereClause = concat (@whereClause,' and ', @condition_temp)
			
		DELETE TOP (1) FROM @conditions
		SELECT @count = COUNT(*) FROM @conditions
	END

	--PAGINATION
	IF @pnum IS NULL AND @size IS NULL
    BEGIN
        SET @paginationClause = ''
    END
    ELSE
    BEGIN
    	EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT
    END

	SET @script= N'
		SELECT
			e.nId_Evaluator,
			e.sEvaluator_Name,
			e.nAnswered,
			COUNT(*) OVER() AS nTotal_Registers
		FROM v_list_evaluators_form_completition_status e
		WHERE e.nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation)+ ' 
			and e.nId_Form_Type = '+ CONVERT(NVARCHAR(MAX), @nId_Form_Type)+ @whereClause +' 
		ORDER BY nId_Evaluator ' + @paginationClause

	EXECUTE sp_executesql @script
END;
GO

CREATE OR ALTER PROCEDURE get_configuraciones(
    @configurations_string nvarchar(max),
    @id_usuario int,
    @id_proyecto int,
    @id_requerimiento int,
    @slug_listado nvarchar(max),
    @Id_Colaborador int = 0
)
AS
BEGIN
    --DECLARE @configurations_string nvarchar(max)= 'empresas.cargos.areas.especialidades.ocupaciones.tipos_colaborador.lideres.directores.tipos_modalidad.tipos_contratacion.tipos_autenticacion.generos.nacionalidades.tipos_persona.estados_civil.tipos_documento.tipos_telefono.codigos_telefono_pais.tipos_email.ubigeos.parentescos.entidades_bancaria.tipos_cuenta_bancaria.sedes.horarios'

    IF @Id_Colaborador = 0
        BEGIN
            SET @Id_Colaborador = (select dbo.function_IdUserORIdColabordor('usuario', @id_usuario))
        END

    DECLARE @configurations_request TABLE
                                    (
                                        sConfiguration nvarchar(max)
                                    )

    INSERT INTO @configurations_request
    SELECT value AS sConfiguration
    FROM STRING_SPLIT(@configurations_string, '.');

    DECLARE @configuraciones TABLE
                             (
                                 sCodigo             nvarchar(max),
                                 sDescripcion        nvarchar(max),
                                 sTipo_Configuracion nvarchar(max)
                             );

    --TIPO DE SERVICIO
    --IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipo_servicio') = 1
    --BEGIN
    --INSERT INTO @configuraciones
    --SELECT
    --c.sCodigo AS sCodigo,
    --c.sDescripcion AS sDescripcion,
    --'TIPO_SERVICIO' AS sTipo_Configuracion
    --FROM
    --Configs c
    --where
    --nEstado = 1
    --and
    --c.sTabla = 'TIPO_SERVICIO';
    --END

    --COORDINADORES DE SUPERVISOR
  IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'coordinadores_supervisor') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT c.nId_Coordinador          AS sCodigo,
                   p.sPersona_Nombre          AS sDescripcion,
                   'COORDINADORES_SUPERVISOR' AS sTipo_Configuracion
            FROM Coordinadores c
                     join Coordinadores_Proyectos cp on cp.nId_Coordinador = c.nId_Coordinador
                     JOIN Personas p ON c.nId_Persona = p.nId_Persona
                     join Supervisor_Proyecto sp on sp.nId_Proyecto = cp.nId_Proyecto
                     join Coordinadores supervisor on supervisor.nId_Coordinador = sp.nId_Supervisor
                     join Personas p_sup on p_sup.nId_Persona = supervisor.nId_Persona
                     join Usuarios u_sup on u_sup.nId_Persona = p_sup.nId_Persona
            WHERE u_sup.nId_Usuario = @id_usuario
              AND c.nEstado = 1;
        END

    --COORDINADOR
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'coordinador') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT c.nId_Coordinador AS sCodigo,
                   p.sPersona_Nombre AS sDescripcion,
                   'COORDINADOR'     AS sTipo_Configuracion
            FROM Coordinadores c
                     JOIN
                 Personas p
     ON
                     c.nId_Persona = p.nId_Persona
            WHERE c.nEstado = 1;
        END


  IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'coordinador_all') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT c.nId_Coordinador AS sCodigo,
                   p.sPersona_Nombre AS sDescripcion,
                   'COORDINADOR_ALL'     AS sTipo_Configuracion
FROM Coordinadores c
                    LEFT JOIN
                 Personas p
     ON
                     c.nId_Persona = p.nId_Persona
        END
    --COORDINADOR TODOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'coordinador_suggest') = 1
        BEGIN
   INSERT INTO @configuraciones
            SELECT c.nId_Coordinador AS sCodigo,
                   p.sPersona_Nombre AS sDescripcion,
                   'COORDINADOR'     AS sTipo_Configuracion
            FROM Coordinadores c
                     JOIN
                 Personas p
                 ON
                     c.nId_Persona = p.nId_Persona
   END

    --ESPECIALIDADES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'especialidades') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Especialidad as sCodigo,
                   sDescripcion     as sDescripcion,
                   'ESPECIALIDADES' as sTipo_Configuracion
            from Especialidades
            where nEstado = 1
        END

    --OCUPACIONES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'ocupaciones') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Ocupacion as sCodigo,
                   sDescripcion  as sDescripcion,
                   'OCUPACIONES' as sTipo_Configuracion
            from Ocupaciones
            where nEstado = 1
        END

    --NACIONALIDADES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'nacionalidades') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Pais         as sCodigo,
                   sGentilicio      as sDescripcion,
                   'NACIONALIDADES' as sTipo_Configuracion
            from Paises
            where nEstado = 1
              and sGentilicio is not null
        END

    --ESTADO_CIVIL
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'estados_civil') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo         as sCodigo,
                   sDescripcion    as sDescripcion,
                   'ESTADOS_CIVIL' as sTipo_Configuracion
            from Configs
            where sTabla = 'ESTADO_CIVIL'
              and nEstado = 1
        END

    -- FECHA MÍNIMA DE REACTIVACIÓN DE COLABORADOR
    IF (select count(sConfiguration)
        from @configurations_request
        where sConfiguration = 'fecha_min_reactivar_colaborador') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT sCodigo                       as sCodigo,
                   sDescripcion                  as sDescripcion,
 'FECHA_REACTIVAR_COLABORADOR' as sTipo_Configuracion
            FROM Configs
            WHERE sTabla = 'FECHA_MIN_REACTIVAR_COLAB'
              and nEstado = 1
        END

    --GENERO
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'generos') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo      as sCodigo,
                   sDescripcion as sDescripcion,
                   'GENEROS'    as sTipo_Configuracion
            from Configs
            where sTabla = 'GENERO'
              and nEstado = 1
        END
      --TIPO_PERSONA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_persona') = 1
        BEGIN
        INSERT INTO @configuraciones
            select sCodigo         as sCodigo,
                   sDescripcion    as sDescripcion,
                   'TIPOS_PERSONA' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_PERSONA'
              and nEstado = 1
      END
    --AREAS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'areas') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Area     as sCodigo,
                   sDescripcion as sDescripcion,
                   'AREAS'      as sTipo_Configuracion
      from Areas
            where nEstado = 1
        END

    --EMPRESAS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'empresas') = 1
        BEGIN
            INSERT INTO @configuraciones
            select empresa.nId_Empresa     as sCodigo,
                   persona.sPersona_Nombre as sDescripcion,
       'EMPRESAS'              as sTipo_Configuracion
  from Empresas_Usuarias empresa
                     left join Personas persona ON empresa.nId_Persona = persona.nId_Persona
        END

    --TIPO_COLABORADOR
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_colaborador') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo             as sCodigo,
                   sDescripcion        as sDescripcion,
                   'TIPOS_COLABORADOR' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_COLABORADOR'
              and nEstado = 1
  END

    --TIPO_CONTRATACION
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_contratacion') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo              as sCodigo,
                   sDescripcion         as sDescripcion,                            'TIPOS_CONTRATACION' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_CONTRATACION'
              and nEstado = 1
        END

    --TIPO_MODALIDAD
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_modalidad') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo           as sCodigo,
                   sDescripcion      as sDescripcion,
                   'TIPOS_MODALIDAD' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_MODALIDAD'
              and nEstado = 1
        END

    --ESTADO_COLABORADOR
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'estados_colaborador') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo               as sCodigo,
                   sDescripcion          as sDescripcion,
                   'ESTADOS_COLABORADOR' as sTipo_Configuracion
            from Configs
          where sTabla = 'ESTADO_COLABORADOR'
              and nEstado = 1
        END

    --CARGOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'cargos') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Cargo    as sCodigo,
                   sNombre as sDescripcion,
                   'CARGOS'     as sTipo_Configuracion
   from Cargos
            where nEstado_Cargos = 1
            --and nId_Cargo in (1,3,4,5,6,7,8,9,10,11,13,14,16,17,18,19,20,21,23,24,25,26,27)--,28//Administrador de Sistemas)
        END

    --LIDERES+DIRECTORES
    if (select count(sConfiguration) from @configurations_request where sConfiguration = 'lideres_directores') = 1
        BEGIN
            INSERT INTO @configuraciones
        SELECT c.nId_Colaborador as sCodigo,
        p.sPersona_Nombre as sDescripcion,
                   CASE
      WHEN r.sDescripcion like '%LÍDER%' OR  r.sDescripcion = 'COORDINADOR' THEN 'LIDER'
                       WHEN r.sDescripcion like '%DIRECTOR%' THEN 'DIRECTOR'
                       END           as sTipo_Configuracion
            FROM Colaboradores c
                     JOIN Personas p ON c.nId_Persona = p.nId_Persona
                     JOIN Usuarios u ON p.nId_Persona = u.nId_Persona
                     JOIN Roles_Usuarios ru ON u.nId_Usuario = ru.nId_Usuario
                     JOIN ROLES r ON ru.nId_Rol = r.nId_Rol
            WHERE r.sDescripcion like '%LÍDER%'
 OR r.sDescripcion like '%DIRECTOR%'
               OR r.sDescripcion = 'COORDINADOR'
    END

    --LIDERES+DIRECTORES-ALL
    if (select count(sConfiguration) from @configurations_request where sConfiguration = 'lideres_directores_all') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT c.nId_Colaborador as sCodigo,
                   p.sPersona_Nombre as sDescripcion,
                   'LIDERES_DIRECTORES' sTipo_Configuracion
            FROM Colaboradores c
                     JOIN Personas p ON c.nId_Persona = p.nId_Persona
                     JOIN Usuarios u ON p.nId_Persona = u.nId_Persona
                     JOIN Roles_Usuarios ru ON u.nId_Usuario = ru.nId_Usuario
   JOIN ROLES r ON ru.nId_Rol = r.nId_Rol
            WHERE c.nEstado_Colaborador = 1
                and r.sDescripcion like '%LÍDER%'
               OR r.sDescripcion like '%DIRECTOR%'
            OR r.sDescripcion = 'COORDINADOR'
        END

  --TIPOS AUTENTICACION
    if (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_autenticacion') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo               as sCodigo,
                   sDescripcion          as sDescripcion,
                   'TIPOS_AUTENTICACION' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_AUTENTICACION'
              and nEstado = 1
        END

    --TIPOS DOCUMENTO
    if (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_documento') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo           as sCodigo,
                   sDescripcion      as sDescripcion,
                   'TIPOS_DOCUMENTO' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_DOCUMENTO'
              and nEstado = 1
        END

    --TIPOS TELEFONO
    if (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_telefono') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo          as sCodigo,
                   sDescripcion     as sDescripcion,
                   'TIPOS_TELEFONO' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_TELEFONO'
              and nEstado = 1
        END

    --CODIGOS TELEFONO PAIS
    if (select count(sConfiguration) from @configurations_request where sConfiguration = 'codigos_telefono_pais') = 1
        BEGIN
   INSERT INTO @configuraciones
            select nId_Pais                                                as sCodigo,
                   CONCAT(sDescripcion, ' (+', sCodigo_Llamada_Inter, ')') as sDescripcion,
                   'CODIGOS_TELEFONO_PAIS'                                 as sTipo_Configuracion
            from Paises
            where nEstado = 1
              and sCodigo_Llamada_Inter is not null
        END

    --TIPOS EMAIL
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_email') = 1
        BEGIN
            INSERT INTO @configuraciones
    select sCodigo       as sCodigo,
             sDescripcion  as sDescripcion,
                   'TIPOS_EMAIL' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_EMAIL'
              and nEstado = 1
        END

    --UBIGEOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'ubigeos') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Ubigeo                                                                  as sCodigo,
                CONCAT(sUbigeo_Departamento, '-', sUbigeo_Provincia, '-', sUbigeo_Distrito) as sDescripcion,
                   'UBIGEOS'                      as sTipo_Configuracion
            from Ubigeos
            where nEstado = 1
   END

  --PARENTESCOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'parentescos') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo       as sCodigo,
                   sDescripcion  as sDescripcion,
                   'PARENTESCOS' as sTipo_Configuracion
            from Configs
            where sTabla = 'PARENTESCO'
         and nEstado = 1
        END

    --ENTIDADES_BANCARIA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'entidades_bancaria') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo              as sCodigo,
                   sDescripcion         as sDescripcion,
 'ENTIDADES_BANCARIA' as sTipo_Configuracion
            from Configs
            where sTabla = 'BANCO'
              and nEstado = 1
        END

    --TIPOS_CUENTA_BANCARIA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_cuenta_bancaria') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo                 as sCodigo,
                   sDescripcion            as sDescripcion,
                   'TIPOS_CUENTA_BANCARIA' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_CUENTA'
              and nEstado = 1
        END

    --SEDES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'sedes') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Sede     as sCodigo,
                   sDescripcion as sDescripcion,
                   'SEDES'      as sTipo_Configuracion
            from Sedes
            where nEstado = 1
              AND sEntity = 'Empresa'
        END

    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'sedes_suggest') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Sede     as sCodigo,
                   sDescripcion as sDescripcion,
                   'SEDES_SUGGEST'      as sTipo_Configuracion
            from Sedes
   where sEntity = 'Empresa'
 AND nEstado = 1
        END

    --COLABORADORES_SIN_CONTRATO
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'colaboradores_sin_contrato') =
       1
        BEGIN
            INSERT INTO @configuraciones
            SELECT CAST(c.nId_Colaborador as nvarchar(max)) as sCodigo,
                   p.sPersona_Nombre                        as sDescripcion,
                   'COLABORADORES_SIN_CONTRATO'             as sTipo_Configuracion
 FROM Colaboradores c
    JOIN Horarios_Colaboradores hc ON c.nId_Colaborador = hc.nId_Colaborador AND hc.nEstado = 1
                     JOIN Personas p ON c.nId_Persona = p.nId_Persona
                     JOIN Usuarios u ON u.nId_Persona = p.nId_Persona
                     JOIN Roles_Usuarios ru ON ru.nId_Usuario = u.nId_Usuario
                   LEFT JOIN Contratos c2 ON c.nId_Colaborador = c2.nId_Colaborador
            WHERE c.nEstado_Colaborador = 1
          AND (
                c2.nId_Contrato is NULL
                )
              AND ru.nId_Rol NOT IN (1, 10)

        END

    --COLABORADORES_CON_CONTRATO
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'colab_contrato_suggest') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT CAST(c.nId_Colaborador as nvarchar(max)) as sCodigo,
                   p.sPersona_Nombre                        as sDescripcion,
                   'COLABORADORES_CON_CONTRATO'             as sTipo_Configuracion
            FROM Colaboradores c
                     JOIN Horarios_Colaboradores hc ON c.nId_Colaborador = hc.nId_Colaborador AND hc.nEstado = 1
                   JOIN Personas p ON c.nId_Persona = p.nId_Persona
           JOIN Usuarios u ON u.nId_Persona = p.nId_Persona
                     JOIN Roles_Usuarios ru ON ru.nId_Usuario = u.nId_Usuario
                     LEFT JOIN Contratos c2 ON c.nId_Colaborador = c2.nId_Colaborador
            WHERE c.nEstado_Colaborador = 1
              AND (
                c2.nId_Contrato IS NOT NULL
                )
 AND ru.nId_Rol NOT IN (1, 10)

      END

    -- COLABORADORES CREACION SOLICITUD MASIVA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'colaboradores_con_usuario') =
       1
        BEGIN
            INSERT INTO @configuraciones
           SELECT DISTINCT CAST (c.nId_Colaborador as nvarchar(max)) as sCodigo,
                   p.sPersona_Nombre                        as sDescripcion,
                   'COLABORADORES_CON_USUARIO' as sTipo_Configuracion
    FROM Colaboradores c
                     JOIN Personas p ON c.nId_Persona = p.nId_Persona
                     JOIN Usuarios u ON u.nId_Persona = p.nId_Persona
                     JOIN Horarios_Colaboradores hc ON c.nId_Colaborador = hc.nId_Colaborador
                     JOIN colaboradores_supervisor cs ON c.nId_Colaborador = cs.nId_Colaborador
            WHERE c.nEstado_Colaborador = 1
              AND hc.nEstado = 1
              --AND c.nId_Colaborador != @Id_Colaborador
              AND cs.bPrincipal = 1
        END

    --COLABORADORES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'colaboradores') = 1
  BEGIN
         IF @slug_listado = 'TODOS' OR @slug_listado = ''
            BEGIN
                    INSERT INTO @configuraciones
                    SELECT CAST(c.nId_Colaborador as nvarchar(max)) as sCodigo,
                           p.sPersona_Nombre                        as sDescripcion,
                           'COLABORADORES'                          as sTipo_Configuracion
                    FROM Colaboradores c
                             JOIN Personas p ON c.nId_Persona = p.nId_Persona
                             JOIN Usuarios u ON u.nId_Persona = p.nId_Persona
                             JOIN Horarios_Colaboradores hc ON c.nId_Colaborador = hc.nId_Colaborador
                    WHERE c.nEstado_Colaborador = 1
                      AND hc.nEstado = 1
                END
            ELSE
 IF @slug_listado = 'EQUIPO'
  BEGIN
            DECLARE @nId_Colaborador int = (SELECT c.nId_Colaborador
                                                        FROM Colaboradores c
                                                                 JOIN personas_colaboradores pc ON pc.nid_colaborador = c.nId_Colaborador
                                                                 JOIN Usuarios u ON u.nId_Persona = pc.nid_persona
                                                        WHERE u.nId_Usuario = @id_usuario)

                DECLARE @Id_Team_Usuario_Lider TABLE
                                                       (
                                                           nId_Team int
                                                       );

                        INSERT INTO @Id_Team_Usuario_Lider
                        SELECT nId_Team
        FROM Team t
          WHERE t.nId_Lider = @nId_Colaborador
                           or t.nId_Director = @nId_Colaborador

                        INSERT INTO @configuraciones
                        SELECT DISTINCT CAST(tc.nId_Colaborador as nvarchar(max)) as sCodigo,
                                        pc.sPersona_Nombre                        as sDescripcion,
                                        'COLABORADORES'                           as sTipo_Configuracion
                        FROM Team_Colaboradors tc
                                 JOIN Colaboradores c on c.nId_Colaborador = tc.nId_Colaborador
                                 JOIN personas_colaboradores pc ON pc.nid_colaborador = c.nId_Colaborador
              WHERE tc.nId_Team IN (select * from @Id_Team_Usuario_Lider)
                          and c.nEstado_Colaborador = 1
                    END
                ELSE
                    IF @slug_listado = 'PROYECTO'
                        BEGIN
       DECLARE @nId_ColaboradorP int = (SELECT c.nId_Colaborador
                                   FROM Colaboradores c
                           JOIN personas_colaboradores pc ON pc.nid_colaborador = c.nId_Colaborador
                                                                      JOIN Usuarios u ON u.nId_Persona = pc.nid_persona
                           WHERE u.nId_Usuario = @id_usuario)

       DECLARE @nId_Proyectos TABLE
          (
                          nId_Proyecto int
                                                   );

                            INSERT INTO @nId_Proyectos
                            SELECT DISTINCT nId_Proyecto
       FROM Proyecto_Lider pl
    WHERE pl.nId_Lider = @nId_ColaboradorP
                              AND pl.nEstado = 1
                            UNION
                            SELECT nId_Proyecto
                            FROM Proyectos p
                            WHERE p.nId_Director = @nId_ColaboradorP

                            INSERT INTO @configuraciones
                            SELECT DISTINCT CAST(c.nId_Colaborador as nvarchar(max)) as sCodigo,
                                            pc.sPersona_Nombre                       as sDescripcion,
                      'COLABORADORES'                          as sTipo_Configuracion
   FROM Colaboradores c
                                     JOIN Horarios_Colaboradores hc ON c.nId_Colaborador = hc.nId_Colaborador
                                     JOIN Colaboradores_Proyectos cp ON cp.nId_Colaborador = c.nId_Colaborador
         JOIN personas_colaboradores pc ON pc.nid_colaborador = c.nId_Colaborador
                            WHERE cp.nId_Proyecto IN (SELECT * FROM @nId_Proyectos)
                              AND hc.nEstado = 1
                              AND c.nEstado_Colaborador = 1
                              AND c.nId_Colaborador <> @nId_ColaboradorP
                        END
        END

  ------ ACTUALIZAR INFORMACION PARA COLABORADRES MASIVOS
  IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'collaborator_self_update_information') = 1
   BEGIN
    INSERT INTO @configuraciones
     select (col.nId_Colaborador) as sCodigo,
     null          as sDescripcion,
     'COLLABORATOR-SELF-UPDATE'  as sTipo_Configuracion
    FROM Colaboradores col
     inner join Usuarios us on us.nId_Persona = col.nId_Persona
    WHERE col.bSelfUpdate = 1 AND us.nId_Usuario = @id_usuario
   END



    --COLABORADORES-PARA SUGERENCIAS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'colaboradores_suggest') = 1
        BEGIN
            INSERT INTO @configuraciones
                EXEC collaborator_suggests_by_slug @slug_listado, @Id_Colaborador;
        END

    --COLABORADORES VISTA CLIENTE
    IF (select count(sConfiguration)
        from @configurations_request
        where sConfiguration = 'colaboradores_de_supervisor') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT DISTINCT CAST(c.nId_Colaborador as nvarchar(max)) as sCodigo,
                            p.sPersona_Nombre                        as sDescripcion,
            'COLABORADORES_DE_SUPERVISOR'            as sTipo_Configuracion
            FROM Colaboradores c
                     JOIN Personas p ON c.nId_Persona = p.nId_Persona
                     JOIN Horarios_Colaboradores hc ON c.nId_Colaborador = hc.nId_Colaborador
            JOIN Colaboradores_Proyectos c_pry ON c_pry.nId_Colaborador = c.nId_Colaborador
                     JOIN Proyectos pry ON pry.nId_Proyecto = c_pry.nId_Proyecto
               JOIN Supervisor_Proyecto sup_pry ON sup_pry.nId_Proyecto = pry.nId_Proyecto
                     JOIN Coordinadores supervisor ON supervisor.nId_Coordinador = sup_pry.nId_Supervisor
    JOIN Personas p_sup ON p_sup.nId_Persona = supervisor.nId_Persona
                     JOIN Usuarios u_sup ON u_sup.nId_Persona = p_sup.nId_Persona
            WHERE c.nEstado_Colaborador = 1
              AND hc.nEstado = 1
              AND u_sup.nId_Usuario = @id_usuario
        END

 --CLIENTES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'clientes') = 1
        BEGIN
            INSERT INTO @configuraciones
            select cliente.nId_Cliente     as sCodigo,
                   persona.sPersona_Nombre as sDescripcion,
             'CLIENTES'              as sTipo_Configuracion
from Clientes cliente
                     left join Personas persona ON cliente.nId_Persona = persona.nId_Persona
            WHERE cliente.nEstado = 1
        END

    --HORARIOS
   IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'horarios') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Horario  as sCodigo,
                   sDescripcion as sDescripcion,
              'HORARIO'    as sTipo_Configuracion
            from Horarios h
            where h.nEstado = 1
        END

    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'horarios_suggest') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Horario  as sCodigo,
                   sDescripcion as sDescripcion,
                   'HORARIO'    as sTipo_Configuracion
            from Horarios h
        END

    --DIAS DE LA SEMANA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'dias_semana') = 1
        BEGIN
            INSERT INTO @configuraciones
        select sCodigo      as sCodigo,
          sDescripcion as sDescripcion,
                   'DIA_SEMANA' as sTipo_Configuracion
            from Configs
            where sTabla = 'DIA_SEMANA'
              and nEstado = 1
        END

    --TIPO MARCACION
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipo_marcacion') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo          as sCodigo,
           sDescripcion     as sDescripcion,
           'TIPO_MARCACION' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_MARCACION'
              and nEstado = 1
        END

    --TIPO ROL (LISTADO DE ROLES CON TIPO)
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipos_rol_interno_externo') =
       1
        BEGIN
            INSERT INTO @configuraciones
            select r.nId_Rol      as sCodigo,
                   r.sDescripcion as sDescripcion,
                   CASE
                       WHEN r.sTipo_Rol = 1 THEN concat('ROL_', c.sDescripcion)
                       WHEN r.sTipo_Rol = 2 THEN concat('ROL_', c.sDescripcion)
                       END        as sTipo_Configuracion
            from Roles r
                     join Configs c on r.sTipo_Rol = c.sCodigo
            and c.sTabla = 'TIPO_ROL'
            where r.nEstado = 1
        END

    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'roles_suggest') = 1
        BEGIN
            INSERT INTO @configuraciones
            select r.nId_Rol      as sCodigo,
                   r.sDescripcion as sDescripcion,
                   'TODO_ROLES'  as sTipo_Configuracion
            from Roles r
                  join Configs c on r.sTipo_Rol = c.sCodigo
                and c.sTabla = 'TIPO_ROL'
            WHERE r.nId_Rol <> 1
        END

    --PERSONAS SIN USUARIO
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'personas_sin_usuario') = 1
        BEGIN
            INSERT INTO @configuraciones
            select p.nId_Persona     as sCodigo,
                   p.sPersona_Nombre as sDescripcion,
                CASE
WHEN (SELECT count(*)
                             from Personas p2
                                      join Colaboradores c on c.nId_Persona = p2.nId_Persona
        where p2.nId_Persona = p.nId_Persona) > 0 THEN 'SIN_USUARIO_COLABORADOR'
                       WHEN (SELECT count(*)
                             from Personas p2
                                      join Coordinadores c2 on c2.nId_Persona = p2.nId_Persona
                             where p2.nId_Persona = p.nId_Persona) > 0 THEN 'SIN_USUARIO_COORDINADOR'
                    END           AS sTipo_Configuracion
            from Personas p
                     left join Usuarios u on p.nId_Persona = u.nId_Persona
where u.nId_Usuario is null
              and p.nId_Tipo_Persona = 1
            and ((SELECT count(*)
                    from Personas p2
                             join Colaboradores c on c.nId_Persona = p2.nId_Persona
                    where p2.nId_Persona = p.nId_Persona) > 0
                or (SELECT count(*)
                    from Personas p2
                             join Coordinadores c2 on c2.nId_Persona = p2.nId_Persona
                    where p2.nId_Persona = p.nId_Persona) > 0)
        END

    --PERSONAS CON USUARIO
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'personas_con_usuario') = 1
        BEGIN
           INSERT INTO @configuraciones
      select p.nId_Persona          as sCodigo,
                   p.sPersona_Nombre      as sDescripcion,
                   'PERSONAS_CON_USUARIO' as sTipo_Configuracion
            from Personas p
                     join Usuarios u on p.nId_Persona = u.nId_Persona
            where p.nEstado = 1
        END

    --TIPO ROL (INTERNO - EXTERNO)
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipo_rol') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sCodigo      as sCodigo,
                   sDescripcion as sDescripcion,
    'TIPO_ROL'   as sTipo_Configuracion
            from Configs
    where sTabla = 'TIPO_ROL'
              and nEstado = 1
        END

    --CATEGORIA TAREA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'categoria_tarea') = 1
        BEGIN
            INSERT INTO @configuraciones
            select c.nId_Categoria   as sCodigo,
                   c.sNombre         as sDescripcion,
                   'CATEGORIA_TAREA' as sTipo_Configuracion
            from Categorias c
            where c.nEstado = 1
        END

    --TIPO ACTIVIDAD
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipo_actividad') = 1
        BEGIN
            declare @isQATeam int = (SELECT count(*)
                                     from Colaboradores_Proyectos cp
                                     where nId_Proyecto = (select nId_Proyecto
                                                 from Proyectos p
                                                           where sNombre = 'ASEGURAMIENTO DE LA CALIDAD')
                                       and nId_Colaborador = (select nId_Colaborador
                                                              from Colaboradores c
                                                                       join Personas p on c.nId_Persona = p.nId_Persona
                                          join Usuarios u on p.nId_Persona = u.nId_Persona
                                                              where u.nId_Usuario = @id_usuario))
            IF @isQATeam > 0
                BEGIN
      INSERT INTO @configuraciones
 select sCodigo  as sCodigo,
                           sDescripcion     as sDescripcion,
                           'TIPO_ACTIVIDAD' as sTipo_Configuracion
    from Configs
                    where sTabla = 'TIPO_ACTIVIDAD'
               and nEstado = 1
    and sDescripcion like '%calidad%'

   END
            ELSE
    BEGIN
                    INSERT INTO @configuraciones
                    select sCodigo          as sCodigo,
                           sDescripcion     as sDescripcion,
                           'TIPO_ACTIVIDAD' as sTipo_Configuracion
                    from Configs
where sTabla = 'TIPO_ACTIVIDAD'
                      and nEstado = 1
        and sDescripcion not like '%calidad%'
                END
        END

    --PROYECTOS DE SUPERVISOR
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'proyectos_supervisor') = 1
        BEGIN
        INSERT INTO @configuraciones
            SELECT *
            FROM (SELECT p.nId_Proyecto         AS sCodigo,
                         CONCAT(p.sNombre, ' ', '(',
                                p2.sPrimer_Nombre,  ')')            AS sDescripcion,
                         'PROYECTOS_SUPERVISOR' AS sTipo_Configuracion
                  FROM Proyectos p
                           JOIN Coordinadores_Proyectos cp ON p.nId_Proyecto = cp.nId_Proyecto
                           JOIN Coordinadores c ON cp.nId_Coordinador = c.nId_Coordinador
                           JOIN Personas p2 ON c.nId_Persona = p2.nId_Persona
                           join Supervisor_Proyecto sp on sp.nId_Proyecto = cp.nId_Proyecto
                           join Coordinadores supervisor on supervisor.nId_Coordinador = sp.nId_Supervisor
                           join Personas p_sup on p_sup.nId_Persona = supervisor.nId_Persona
                           join Usuarios u_sup on u_sup.nId_Persona = p_sup.nId_Persona
                  WHERE u_sup.nId_Usuario = @id_usuario)
                     AS proyectos
            WHERE sDescripcion NOT LIKE '%SIN PROYECTO%'
        END

    --PROYECTOS COORDINADOR
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'proyectos_coor') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT p.nId_Proyecto   AS sCodigo,
                   CONCAT(p.sNombre, ' ', '(',
                          p_coor.sPrimer_Nombre ,
                          ')')      AS sDescripcion,
                   'PROYECTOS_COOR' AS sTipo_Configuracion
            FROM Proyectos p
                     JOIN Coordinadores_Proyectos cp ON p.nId_Proyecto = cp.nId_Proyecto
                     JOIN Coordinadores c ON cp.nId_Coordinador = c.nId_Coordinador
                     join Personas p_coor on p_coor.nId_Persona = c.nId_Persona
                     join Usuarios u_coor on u_coor.nId_Persona = p_coor.nId_Persona
            WHERE u_coor.nId_Usuario = @id_usuario
          AND c.nEstado = 1;
END

 -- cambio DAVID (todos los proyectos)
 IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'proyectos_todos_masivo') = 1
        BEGIN
            INSERT INTO @configuraciones
            select *
            from ((select p.nId_Proyecto  as sCodigo,
                  CONCAT(p.sNombre, ' ', '(', p2.sPrimer_Nombre , ')') as sDescripcion,
                          'PROYECTOS_TODOS_MASIVO' as sTipo_Configuracion
                   from Proyectos p
                            join Coordinadores_Proyectos cp on p.nId_Proyecto = cp.nId_Proyecto
     join Coordinadores c on cp.nId_Coordinador = c.nId_Coordinador
                            join Personas p2 on c.nId_Persona = p2.nId_Persona
                   )
                  union
                  (SELECT p.nId_Proyecto         AS sCodigo,
        CONCAT(
             p.sCodigo ,' ',p.sNombre, ' ','(',p2.sPrimer_Nombre ,')'
           )               AS sDescripcion,
                          'PROYECTOS_CON_ACCESO' AS sTipo_Configuracion
                   FROM Proyectos p
                            JOIN Coordinadores_Proyectos cp ON p.nId_Proyecto = cp.nId_Proyecto
                            JOIN Coordinadores c ON cp.nId_Coordinador = c.nId_Coordinador
                 JOIN Personas p2 ON c.nId_Persona = p2.nId_Persona
                   WHERE p.nEstado = 1
                AND (
                       p.nTipo_Acceso = 0
                           OR (
                           p.nTipo_Acceso = 1
                               AND EXISTS (SELECT 1
                                           FROM Colaboradores_Proyectos cop
                                           WHERE cop.nId_Proyecto = p.nId_Proyecto
                                             AND cop.nId_Colaborador = @Id_Colaborador)
                               OR EXISTS (SELECT 1
                                          FROM Proyecto_Lider pl
                                          WHERE pl.nId_Proyecto = p.nId_Proyecto
                                            AND pl.nId_Lider = @Id_Colaborador)
                           )
                       ))

                  UNION

                  (select p.nId_Proyecto         as sCodigo,
                          CONCAT(p.sNombre, ' ', '(', p3.sPrimer_Nombre , ')') as sDescripcion,
                          'PROYECTOS_COLABORADOR'                               as sTipo_Configuracion
                   from Proyectos p
                            join Coordinadores_Proyectos cp on p.nId_Proyecto = cp.nId_Proyecto
                          join Coordinadores c on cp.nId_Coordinador = c.nId_Coordinador
                            join Personas p2 on c.nId_Persona = p2.nId_Persona
                            join Colaboradores_Proyectos cp2 on p.nId_Proyecto = cp2.nId_Proyecto
                            join Colaboradores c2 on cp2.nId_Colaborador = c2.nId_Colaborador
                     join Personas p3 on c2.nId_Persona = p3.nId_Persona
                            join Usuarios u on p3.nId_Persona = u.nId_Persona
                   where u.nId_Usuario = @id_usuario
                     and p.nEstado = 1)
                  union
                  (select p.nId_Proyecto                                        as sCodigo,
                          CONCAT(p.sNombre, ' ', '(', p2.sPrimer_Nombre , ')') as sDescripcion,
                          'PROYECTOS_LIDER_DIRECTOR'                            as sTipo_Configuracion
                   from Proyectos p
                            join Proyecto_Lider pl on p.nId_Proyecto = pl.nId_Proyecto and pl.nEstado = 1--PRY_LIDER
                            join Coordinadores_Proyectos cp on p.nId_Proyecto = cp.nId_Proyecto
           join Coordinadores c on cp.nId_Coordinador = c.nId_Coordinador
join Personas p2 on c.nId_Persona = p2.nId_Persona where (p.nId_Lider = @Id_Colaborador
                       or pl.nId_Lider = @Id_Colaborador)
                     and p.nEstado = 1)
                  union
                  (select p.nId_Proyecto                                        as sCodigo,
                          CONCAT(p.sNombre, ' ', '(', p_coor.sPrimer_Nombre , ')') as sDescripcion,
                          'PROYECTOS_COORDINADOR'                               as sTipo_Configuracion
                   from Proyectos p
                            join Coordinadores_Proyectos cp on cp.nId_Proyecto = p.nId_Proyecto
            join Coordinadores coo on coo.nId_Coordinador = cp.nId_Coordinador
                            join Personas p_coor on p_coor.nId_Persona = coo.nId_Persona
                            join Usuarios u on u.nId_Persona = p_coor.nId_Persona
                   where u.nId_Usuario = @id_usuario
                     and p.nEstado = 1)) as proyectos
            where sDescripcion not like '%SIN PROYECTO%'
        END
 -- cambio DAVID (todos los proyectos)

    --PROYECTOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'proyectos') = 1
        BEGIN
        INSERT INTO @configuraciones
            select *
            from ((select p.nId_Proyecto                                        as sCodigo,
                          CONCAT(p.sNombre, ' ', '(', p2.sPrimer_Nombre, ')') as sDescripcion,
 'PROYECTOS'                   as sTipo_Configuracion
                   from Proyectos p
                            join Coordinadores_Proyectos cp on p.nId_Proyecto = cp.nId_Proyecto
                            join Coordinadores c on cp.nId_Coordinador = c.nId_Coordinador
                            join Personas p2 on c.nId_Persona = p2.nId_Persona

                   where p.nEstado = 1)
                  union
                  (SELECT p.nId_Proyecto         AS sCodigo,

                          CONCAT(
                          p.sNombre, ' ',
                                  '(',
                                    p2.sPrimer_Nombre,
                                  ')'
                          )                      AS sDescripcion,
         'PROYECTOS_CON_ACCESO' AS sTipo_Configuracion
                   FROM Proyectos p
                            JOIN Coordinadores_Proyectos cp ON p.nId_Proyecto = cp.nId_Proyecto
                            JOIN Coordinadores c ON cp.nId_Coordinador = c.nId_Coordinador
                            JOIN Personas p2 ON c.nId_Persona = p2.nId_Persona
                            LEFT JOIN Configs c1 ON c1.sTabla = 'PREFIJO_PROYECTO'
                                  AND c1.sCodigo = CASE
                                                   WHEN p.sPrefijo = 1 THEN '1'
                                                   WHEN p.sPrefijo = 2 THEN '2'
               ELSE NULL
                                                   END

                   WHERE p.nEstado = 1
                     AND (p.nTipo_Acceso = 0
    OR (
                           p.nTipo_Acceso = 1
        AND EXISTS (SELECT 1
                                           FROM Colaboradores_Proyectos cop
                                     WHERE cop.nId_Proyecto = p.nId_Proyecto
                                             AND cop.nId_Colaborador = @Id_Colaborador)
                               OR EXISTS (SELECT 1
                                          FROM Proyecto_Lider pl
              WHERE pl.nId_Proyecto = p.nId_Proyecto
                           AND pl.nId_Lider = @Id_Colaborador)
                           )
                       ))

                  UNION

                  (select p.nId_Proyecto                                        as sCodigo,
                          CONCAT(p.sNombre, ' ', '(', p2.sPrimer_Nombre, ')') as sDescripcion,
                          'PROYECTOS_COLABORADOR'                               as sTipo_Configuracion
                   from Proyectos p
                            join Coordinadores_Proyectos cp on p.nId_Proyecto = cp.nId_Proyecto
                            join Coordinadores c on cp.nId_Coordinador = c.nId_Coordinador
                            join Personas p2 on c.nId_Persona = p2.nId_Persona
                            join Colaboradores_Proyectos cp2 on p.nId_Proyecto = cp2.nId_Proyecto
                            join Colaboradores c2 on cp2.nId_Colaborador = c2.nId_Colaborador
                            join Personas p3 on c2.nId_Persona = p3.nId_Persona
                            join Usuarios u on p3.nId_Persona = u.nId_Persona
                   where u.nId_Usuario = @id_usuario
                     and p.nEstado = 1)
                  union
                  (select p.nId_Proyecto           as sCodigo,
           CONCAT(p.sNombre, ' ', '(', p2.sPrimer_Nombre , ')') as sDescripcion,
                          'PROYECTOS_LIDER_DIRECTOR'                  as sTipo_Configuracion
             from Proyectos p
                      join Proyecto_Lider pl on p.nId_Proyecto = pl.nId_Proyecto and pl.nEstado = 1--PRY_LIDER
                            join Coordinadores_Proyectos cp on p.nId_Proyecto = cp.nId_Proyecto
                            join Coordinadores c on cp.nId_Coordinador = c.nId_Coordinador
                            join Personas p2 on c.nId_Persona = p2.nId_Persona
                   where (p.nId_Lider = @Id_Colaborador
                       or pl.nId_Lider = @Id_Colaborador)
                     and p.nEstado = 1)
                  union
                  (select p.nId_Proyecto                                        as sCodigo,
                          CONCAT(p.sNombre, ' ', '(', p_coor.sPrimer_Nombre, ')') as sDescripcion,
                          'PROYECTOS_COORDINADOR' as sTipo_Configuracion
                   from Proyectos p
                            join Coordinadores_Proyectos cp on cp.nId_Proyecto = p.nId_Proyecto
           join Coordinadores coo on coo.nId_Coordinador = cp.nId_Coordinador
                            join Personas p_coor on p_coor.nId_Persona = coo.nId_Persona
                            join Usuarios u on u.nId_Persona = p_coor.nId_Persona
                   where u.nId_Usuario = @id_usuario
                     and p.nEstado = 1)) as proyectos
            where sDescripcion not like '%SIN PROYECTO%'
      END


    -- PRY: todo: see if it has to replace PROYECTOS_COLABORADOR
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'proyectos_colaborador_v2') = 1
        BEGIN
        INSERT INTO @configuraciones                SELECT p.nId_Proyecto             as sCodigo,
                   CONCAT(p.sNombre, ' ', '(', pc.sPrimer_Nombre , ')') as sDescripcion,
                   'PROYECTOS_COLABORADOR_V2'                            as sTipo_Configuracion
            FROM Proyectos p
                     JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto
                     JOIN Coordinadores coor ON coor.nId_Coordinador = cp.nId_Coordinador
                     JOIN Personas pc ON pc.nId_Persona = coor.nId_Persona
            WHERE p.nId_Proyecto IN (SELECT t.nId_Proyecto
             FROM Tareas t
              WHERE t.nId_Colaborador = @Id_Colaborador)
        END

    -- RQ
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'requerimientos_colaborador') =
       1
        BEGIN
            INSERT INTO @configuraciones
            SELECT r.nId_Requerimiento          as sCodigo,
            r.sNombre                    as sDescripcion,
                   'REQUERIMIENTOS_COLABORADOR' as sTipo_Configuracion
            FROM Requerimientos r
            WHERE r.nId_Proyecto IN (SELECT t.nId_Proyecto
                                     FROM Tareas t
                                     WHERE t.nId_Colaborador = @Id_Colaborador)
        END

    -- PRY-RQ COMPOSED FILTER
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'proyectos_todos') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT p.nId_Proyecto    as sCodigo,
                   p.sNombre         as sDescripcion,
                   'PROYECTOS_TODOS' as sTipo_Configuracion
            FROM Proyectos p
            WHERE sDescripcion NOT LIKE '%SIN PROYECTO%'
        END

    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'pry_req_colaborador') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT '0'            as sCodigo,
CONCAT(r.nId_Proyecto, '-', r.nId_Requerimiento) as sDescripcion,
                   'PRY_REQ_COLABORADOR'                            as sTipo_Configuracion
            FROM Requerimientos r
            WHERE r.nId_Proyecto IN (SELECT t.nId_Proyecto
                     FROM Tareas t
                         WHERE t.nId_Colaborador = @Id_Colaborador)

        END

    -- BY PROJECT:
    IF (SELECT COUNT(sConfiguration)
        FROM @configurations_request
        WHERE sConfiguration = 'requerimientos_por_proyecto') = 1
        BEGIN
            INSERT INTO @configuraciones
   SELECT r.nId_Requerimiento          as sCodigo,
          r.sNombre                     as sDescripcion,
            'REQUERIMIENTOS_POR_PROYECTO' as sTipo_Configuracion
            FROM Proyectos p
                     JOIN Requerimientos r ON r.nId_Proyecto = p.nId_Proyecto
            WHERE p.nId_Proyecto IN (SELECT p.nId_Proyecto
                         FROM Proyectos p
                                              JOIN Proyecto_Lider pl ON pl.nId_Proyecto = p.nId_Proyecto
                                     WHERE p.nId_Lider = @Id_Colaborador
                                       and pl.nId_Lider = @Id_Colaborador
                                       and pl.nEstado = 1)
        END

    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'pry_req_por_proyecto') = 1
        BEGIN
            INSERT INTO @configuraciones
     SELECT '0'                                        as sCodigo,
            CONCAT(p.nId_Proyecto, '-', r.nId_Requerimiento) as sDescripcion,
                   'PRY_REQ_POR_PROYECTO'     as sTipo_Configuracion
            FROM Proyectos p
                     JOIN Requerimientos r ON r.nId_Proyecto = p.nId_Proyecto
            WHERE p.nId_Proyecto IN (SELECT p.nId_Proyecto
                                     FROM Proyectos p
                                              JOIN Proyecto_Lider pl ON pl.nId_Proyecto = p.nId_Proyecto
                                     WHERE p.nId_Lider = @Id_Colaborador
                                       and pl.nId_Lider = @Id_Colaborador
                                       and pl.nEstado = 1)
        END

-- BY TEAM:
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'proyectos_team') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT p.nId_Proyecto                                        as sCodigo,
                   CONCAT(p.sNombre, ' ', '(', pc.sPrimer_Nombre, ')') as sDescripcion,
                   'PROYECTOS_TEAM'                                      as sTipo_Configuracion
            FROM Proyectos p
                     JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto
                     JOIN Coordinadores coor ON coor.nId_Coordinador = cp.nId_Coordinador
                     JOIN Personas pc ON pc.nId_Persona = coor.nId_Persona
            WHERE p.nId_Proyecto IN (SELECT tar.nId_Proyecto
                                     FROM Team t
                                              JOIN Team_Colaboradors tc ON t.nId_Team = tc.nId_Team
          JOIN Tareas tar ON tar.nId_Colaborador = tc.nId_Colaborador
                                     WHERE t.nId_Lider = @Id_Colaborador
                                        OR t.nId_Director = @Id_Colaborador)
        END

    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'requerimientos_team') = 1
        BEGIN
            INSERT INTO @configuraciones
   SELECT r.nId_Requerimiento   as sCodigo,
          r.sNombre             as sDescripcion,
                   'REQUERIMIENTOS_TEAM' as sTipo_Configuracion
            FROM Proyectos p
                     JOIN Requerimientos r ON r.nId_Proyecto = p.nId_Proyecto
    WHERE r.nId_Proyecto IN (SELECT tar.nId_Proyecto
             FROM Team t
            JOIN Team_Colaboradors tc ON t.nId_Team = tc.nId_Team
                                              JOIN Tareas tar ON tar.nId_Colaborador = tc.nId_Colaborador
                                     WHERE t.nId_Lider = @Id_Colaborador
                     OR t.nId_Director = @Id_Colaborador)
END

    IF (SELECT COUNT(sConfiguration)
        FROM @configurations_request
        WHERE sConfiguration = 'proyectos_requerimientos_team') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT '0'                                              as sCodigo,
             CONCAT(p.nId_Proyecto, '-', r.nId_Requerimiento) as sDescripcion,
                   'PROYECTOS_REQUERIMIENTOS_TEAM'                  as sTipo_Configuracion
            FROM Proyectos p
                     JOIN Requerimientos r ON r.nId_Proyecto = p.nId_Proyecto
            WHERE r.nId_Proyecto IN (SELECT tar.nId_Proyecto
                                     FROM Team t
                                              JOIN Team_Colaboradors tc ON t.nId_Team = tc.nId_Team
          JOIN Tareas tar ON tar.nId_Colaborador = tc.nId_Colaborador
    WHERE t.nId_Lider = @Id_Colaborador
                       OR t.nId_Director = @Id_Colaborador)
        END

    -- BY ALL:
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'proyectos_requerimientos') = 1
        BEGIN
           INSERT INTO @configuraciones
            SELECT '0'                 as sCodigo,
                   CONCAT(p.nId_Proyecto, '-', r.nId_Requerimiento) as sDescripcion,
           'PROYECTOS_REQUERIMIENTOS'                       as sTipo_Configuracion
            FROM Proyectos p
                     JOIN Requerimientos r ON r.nId_Proyecto = p.nId_Proyecto
        END

    -- BY PROJECT AND TEAM:
    -- projects:
    IF (SELECT COUNT(sConfiguration)
        FROM @configurations_request
        WHERE sConfiguration = 'proyectos_en_equipo_proyecto') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT DISTINCT p.nId_Proyecto                                        as sCodigo,
                            CONCAT(p.sNombre, ' ', '(', p2.sPrimer_Nombre, ')') as sDescripcion,
                            'PROYECTOS_EN_EQUIPO_PROYECTO'                        as sTipo_Configuracion
            FROM Proyectos p
                     JOIN Proyecto_Lider pl ON p.nId_Proyecto = pl.nId_Proyecto AND pl.nEstado = 1 --PRY_LIDER

            -- coordinator name
                     JOIN Coordinadores_Proyectos cp ON p.nId_Proyecto = cp.nId_Proyecto
                     JOIN Coordinadores c ON cp.nId_Coordinador = c.nId_Coordinador
                     JOIN Personas p2 ON c.nId_Persona = p2.nId_Persona
            WHERE ((p.nId_Lider = @Id_Colaborador OR pl.nId_Lider = @Id_Colaborador) AND
                   p.nEstado = 1) -- leader's project
               OR p.nId_Proyecto IN ( -- leader's tasks project
                SELECT tar.nId_Proyecto
     FROM Team t
                         JOIN Team_Colaboradors tc ON t.nId_Team = tc.nId_Team
             JOIN Tareas tar ON tar.nId_Colaborador = tc.nId_Colaborador
  WHERE t.nId_Lider = @Id_Colaborador
                   OR t.nId_Director = @Id_Colaborador)
        END

    -- -- requirements:
    IF (SELECT COUNT(sConfiguration)
        FROM @configurations_request
        WHERE sConfiguration = 'requerimientos_en_equipo_proyecto') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT r.nId_Requerimiento                 as sCodigo,
                   r.sNombre             as sDescripcion,
                   'REQUERIMIENTOS_EN_EQUIPO_PROYECTO' as sTipo_Configuracion
            FROM Proyectos p
                     JOIN Requerimientos r ON r.nId_Proyecto = p.nId_Proyecto
            WHERE r.nId_Proyecto IN (SELECT tar.nId_Proyecto
                                     FROM Team t
                                              JOIN Team_Colaboradors tc ON t.nId_Team = tc.nId_Team
                             JOIN Tareas tar ON tar.nId_Colaborador = tc.nId_Colaborador
                                     WHERE t.nId_Lider = @Id_Colaborador
      OR t.nId_Director = @Id_Colaborador)
OR p.nId_Proyecto IN (SELECT p.nId_Proyecto
          FROM Proyectos p
                                              JOIN Proyecto_Lider pl ON pl.nId_Proyecto = p.nId_Proyecto and pl.nEstado = 1
                                     WHERE p.nId_Lider = @Id_Colaborador)
        END

    -- -- projects and requirements:
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'pry_req_en_equipo_proyecto') =
       1
        BEGIN
            INSERT INTO @configuraciones
         SELECT '0'                                              as sCodigo,
                   CONCAT(p.nId_Proyecto, '-', r.nId_Requerimiento) as sDescripcion,
                   'PRY_REQ_EN_EQUIPO_PROYECTO'            as sTipo_Configuracion
            FROM Proyectos p
                JOIN Requerimientos r ON r.nId_Proyecto = p.nId_Proyecto
            WHERE r.nId_Proyecto IN (SELECT tar.nId_Proyecto
                                     FROM Team t
                                              JOIN Team_Colaboradors tc ON t.nId_Team = tc.nId_Team
                      JOIN Tareas tar ON tar.nId_Colaborador = tc.nId_Colaborador
                                     WHERE t.nId_Lider = @Id_Colaborador
                                        OR t.nId_Director = @Id_Colaborador)
               OR p.nId_Proyecto IN (SELECT p.nId_Proyecto
                                     FROM Proyectos p
                                              JOIN Proyecto_Lider pl ON pl.nId_Proyecto = p.nId_Proyecto and pl.nEstado = 1
                                     WHERE p.nId_Lider = @Id_Colaborador)
        END
    ----------------------------------------------------------------------------------------

    --PROYECTOS SUGERENCIAS
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'proyectos_suggest') = 1                  BEGIN
            IF @slug_listado = 'TODOS'
                BEGIN
                    INSERT INTO @configuraciones
                    SELECT p.nId_Proyecto                       AS sCodigo,
                           p.sNombre     AS sDescripcion,
                           'PROYECTOS_SUGERENCIAS' AS sTipo_Configuracion
                   FROM Proyectos p
                    GROUP BY p.sNombre, p.sCodigo, p.nId_Proyecto
                END
            ELSE
 IF @slug_listado = 'PROYECTO'
                    BEGIN
 INSERT INTO @configuraciones
                        SELECT 1                       AS sCodigo,
 p.sNombre     AS sDescripcion,
        'PROYECTOS_SUGERENCIAS' AS sTipo_Configuracion
         FROM Proyectos p
   JOIN Requerimientos r on p.nId_Proyecto = r.nId_Proyecto
   JOIN Proyecto_Lider pl on pl.nId_Proyecto = p.nId_Proyecto
   WHERE (p.nId_Director = @Id_Colaborador or pl.nId_Lider = @Id_Colaborador)
                          and pl.nEstado = 1
                        GROUP by p.nId_Proyecto,
                                 p.sNombre , p.sCodigo
                    END
                ELSE
                    IF @slug_listado = 'EQUIPO'
                BEGIN
                            INSERT INTO @configuraciones
                            SELECT 1                       AS sCodigo,
                                   p.sNombre     AS sDescripcion,
                       'PROYECTOS_SUGERENCIAS' AS sTipo_Configuracion
                            FROM Proyectos p
                                     JOIN Colaboradores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto
                                     JOIN Team_Colaboradors tc ON tc.nId_Colaborador = cp.nId_Colaborador
                              JOIN Team t ON t.nId_Team = tc.nId_Team
                                     JOIN personas_colaboradores pc ON pc.nid_colaborador = t.nId_Lider
           JOIN Usuarios u ON u.nId_Persona = pc.nid_persona
                            WHERE u.nId_Usuario = @id_usuario
   GROUP BY p.sNombre, p.sCodigo
END
        END

    --REQUERIMIENTOS SUGERENCIAS
IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'requerimientos_suggest') = 1
        BEGIN
            IF @slug_listado = 'TODOS'
                BEGIN
                    INSERT INTO @configuraciones
                    SELECT 1                            AS sCodigo,
                           r.sNombre     AS sDescripcion,
                           'REQUERIMIENTOS_SUGERENCIAS' AS sTipo_Configuracion
                    FROM Requerimientos r
                    GROUP BY r.sNombre , r.sCodigo
                END
            ELSE
                IF @slug_listado = 'PROYECTO'
                    BEGIN
                        INSERT INTO @configuraciones
                        SELECT 1                            AS sCodigo,
                                r.sNombre     AS sDescripcion,
                               'REQUERIMIENTOS_SUGERENCIAS' AS sTipo_Configuracion
                        from Requerimientos r
                        where nId_Lider = @Id_Colaborador
                        GROUP by sNombre , sCodigo;

                    END
                ELSE
                    IF @slug_listado = 'EQUIPO'
                        BEGIN
                            INSERT INTO @configuraciones
                            SELECT 1                            AS sCodigo,
                                    r.sNombre     AS sDescripcion,
                                   'REQUERIMIENTOS_SUGERENCIAS' AS sTipo_Configuracion
                            FROM Requerimientos r
                                     JOIN Proyectos p ON p.nId_Proyecto = r.nId_Proyecto
                                     JOIN Colaboradores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto
                                     JOIN Team_Colaboradors tc ON tc.nId_Colaborador = cp.nId_Colaborador
                                     JOIN Team t ON t.nId_Team = tc.nId_Team
                                     JOIN personas_colaboradores pc ON pc.nid_colaborador = t.nId_Lider
                  JOIN Usuarios u ON u.nId_Persona = pc.nid_persona
                   WHERE u.nId_Usuario = @id_usuario
          GROUP BY r.sNombre, r.sCodigo
                        END
     END

    --TIPOS DE HORAS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipo_hora') = 1
        BEGIN
         INSERT INTO @configuraciones
            select sCodigo      as sCodigo,
         sDescripcion as sDescripcion,
                   'TIPO_HORA'  as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_HORA'
              and nEstado = 1
        END

    --TIPOS DE SERVICIOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipo_servicio') = 1
        BEGIN
            INSERT INTO @configuraciones
       select sCodigo         as sCodigo,
                   sDescripcion    as sDescripcion,
                   'TIPO_SERVICIO' as sTipo_Configuracion
            from Configs
            where sTabla = 'TIPO_SERVICIO'
        END

    --CLIENTES-SERVICIOS-PROYECTOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'cliente_servicio_proyecto') =
       1
        BEGIN
            INSERT INTO @configuraciones
            select '0'                 as sCodigo,
       CONCAT(c.nId_Cliente, '-', p.sId_Tipo_Servicio, '-', p.nId_Proyecto) as sDescripcion,
              'CLIENTE_SERVICIO_PROYECTO'                       as sTipo_Configuracion
            from Proyectos p
                     join Coordinadores_Proyectos cp on p.nId_Proyecto = cp.nId_Proyecto
                     join Coordinadores c on cp.nId_Coordinador = c.nId_Coordinador
        END

    --TIPO_MOTIVOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipo_motivo') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT nId_Tipo_Motivo                             sCodigo,
     CONCAT(tm.sDescripcion, '-', tm.nRecuperable) as sDescripcion,
                   'TIPO_MOTIVO'                                    sTipo_Configuracion
            from Tipos_Motivos as tm
            where nEstado = 1
        END

    --TIPOS_SOLICITUD
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'tipo_solicitud') = 1
        BEGIN
         INSERT INTO @configuraciones
            SELECT nId_Tipo_Solicitud    AS sCodigo,
                   concat(
                           sDescripcion, '-', nId_Categoria) AS sDescripcion,
                   'TIPO_SOLICITUD'                          AS sTipo_Configuracion
            from Tipos_Solicitudes
            where nEstado = 1
		    --AND nId_Tipo_Solicitud NOT IN (26)
             -- AND nId_Tipo_Solicitud NOT IN (21, 20)
        END

    --FERIADOS_NOLABORALES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'feriados') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT nId_Feriado sCodigo,
                   dFecha      sDescripcion,
                   'FERIADOS'  nTipo_Feriado
            from Feriados
           where nEstado = 1
        END

    --FRONTEND_VERSION
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'frontend_version') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT TOP 1
                   nid_v       sCodigo,
                   sVersion       sDescripcion,
                   'FRONTEND_VERSION' sTipo_Configuracion
  from Version
            where nEstado = 1
              and sNombre_Version = 'FRONTEND_VERSION'
        END

    --USUARIOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'usuarios') = 1
        BEGIN
            INSERT INTO @configuraciones
            select u.nId_Usuario     as sCodigo,
                   p.sPersona_Nombre as sDescripcion,
                   'USUARIOS'        as sTipo_Configuracion
            from Usuarios u
                     join Personas p on u.nId_Persona = p.nId_Persona
            where u.nEstado = 1
        END

    --USUARIOS SUGERENCIAS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'usuarios_suggest') = 1
        BEGIN
            INSERT INTO @configuraciones
    select u.nId_Usuario     as sCodigo,
                   p.sPersona_Nombre as sDescripcion,
                 'USUARIOS'        as sTipo_Configuracion
    from Usuarios u
                     join Personas p on u.nId_Persona = p.nId_Persona
            where u.nEstado = 1
              and p.sPersona_Nombre <> 'Smart'
   and p.sPersona_Nombre NOT LIKE '%ADM - %'
        END

    --EXTEMPORANEA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'extemporanea') = 1
        BEGIN
            INSERT INTO @configuraciones
                exec sp_marca_extemporanea_disponible @id_usuario
        END

 -- requerimientos david xd:
 if (select count(sConfiguration) from @configurations_request where sConfiguration = 'requerimientos_activos_inactivos') = 1
 begin
  INSERT INTO @configuraciones
  SELECT r.nId_Requerimiento as sCodigo,
     r.sNombre     AS sDescripcion,
    'REQUERIMIENTOS_ACTIVOS_INACTIVOS'     as sTipo_Configuracion
  FROM Requerimientos r
  WHERE r.nId_Proyecto = @id_proyecto
 end


    --TIPO_REQUERIMIENTO
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'requerimientos') = 1
     BEGIN
            INSERT INTO @configuraciones
            SELECT r.nId_Requerimiento as sCodigo,
                  r.sNombre    as sDescripcion,
                 'REQUERIMIENTO'     as sTipo_Configuracion
            FROM Requerimientos r
  WHERE r.nId_Proyecto = @id_proyecto
       and r.nEstado = 1
        END

    --EQUIPOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'equipos') = 1
        BEGIN
            INSERT INTO @configuraciones
            select t.nId_Team        as sCodigo,
                   CONCAT(p_lider.sPrimer_Nombre, ' ', p_lider.sApe_Paterno, ', ', p_director.sPrimer_Nombre, ' ',
                          p_director.sApe_Paterno) as sDescripcion,
                   'EQUIPOS'                       as sTipo_Configuracion
            from Team t
                     join Colaboradores c_lider on t.nId_Lider = c_lider.nId_Colaborador
                     join Personas p_lider on c_lider.nId_Persona = p_lider.nId_Persona
                     join Colaboradores c_director on t.nId_Director = c_director.nId_Colaborador
                     join Personas p_director on c_director.nId_Persona = p_director.nId_Persona
            WHERE t.nEstado = 1
        END

    --SUPERVISORES  TODOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'supervisores_todos') = 1
        BEGIN
            INSERT INTO @configuraciones
   SELECT NULL AS sCodigo,
          'AUTOGESTIONADO' AS sDescripcion,
          'SUPERVISORES_TODOS' AS sTipo_Configuracion
   UNION
   SELECT CAST(c.nId_Colaborador AS VARCHAR(255)) AS sCodigo,
          p_Lider.sPersona_Nombre AS sDescripcion,
          'SUPERVISORES_TODOS' AS sTipo_Configuracion
   FROM Colaboradores c
   JOIN Personas p_lider ON c.nId_Persona = p_lider.nId_Persona
   WHERE c.nEstado_Colaborador = 1 AND c.nEstado = 1
   AND p_Lider.sPersona_Nombre NOT LIKE '%ADM -%';
            --AND t.nEstado_Team = 1
        END

    --SUPERVISORES_ASIGNADOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'supervisores_asignados') = 1
        BEGIN
            INSERT INTO @configuraciones
            select sa.nId_Supervisor          as sCodigo,
                   p_lider.sPersona_Nombre    as sDescripcion,
                   'SUPERVISORES_ASIGNADOS'   as sTipo_Configuracion
            from   v_Supervisores_Asginados sa
            join Colaboradores c_lider on sa.nId_Supervisor  = c_Lider.nId_Colaborador
            join Personas p_lider on c_lider.nId_Persona = p_lider.nId_Persona
        END


    --EQUIPOS POR LIDER
  IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'equipos_lider') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT t.nId_Team                      as sCodigo,
                   CONCAT(p_lider.sPrimer_Nombre, ' ', p_lider.sApe_Paterno, ', ', p_director.sPrimer_Nombre, ' ',
                          p_director.sApe_Paterno) as sDescripcion,
                   'EQUIPOS_LIDER'                 as sTipo_Configuracion
            FROM Team t
                     join Colaboradores c_lider on t.nId_Lider = c_lider.nId_Colaborador
                     join Personas p_lider on c_lider.nId_Persona = p_lider.nId_Persona
                     join Colaboradores c_director on t.nId_Director = c_director.nId_Colaborador
                     join Personas p_director on c_director.nId_Persona = p_director.nId_Persona
            WHERE t.nId_Lider = @Id_Colaborador
               OR t.nId_Director = @Id_Colaborador
                AND t.nEstado = 1
        END

    --EQUIPOS SUPERVISOR
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'equipos_supervisor') = 1
        BEGIN
            INSERT INTO @configuraciones
            select DISTINCT CONCAT(pry.nId_Lider, '/', pry.nId_Director) as sCodigo,
            CONCAT(p_lider.sPrimer_Nombre, ' ', p_lider.sApe_Paterno, ', ', p_director.sPrimer_Nombre,
                     ' ', p_director.sApe_Paterno)         as sDescripcion,
      'EQUIPOS_SUPERVISOR'                         as sTipo_Configuracion
            from Proyectos pry
                     join Colaboradores c_lider on pry.nId_Lider = c_lider.nId_Colaborador
                     join Personas p_lider on c_lider.nId_Persona = p_lider.nId_Persona
                     join Colaboradores c_director on pry.nId_Director = c_director.nId_Colaborador
                     join Personas p_director on c_director.nId_Persona = p_director.nId_Persona

                     join Coordinadores_Proyectos coor_pry on coor_pry.nId_Proyecto = pry.nId_Proyecto
                     join Supervisor_Proyecto sup_pry on sup_pry.nId_Proyecto = coor_pry.nId_Proyecto

                     join Coordinadores supervisor on supervisor.nId_Coordinador = sup_pry.nId_Supervisor
         join Personas p_sup on p_sup.nId_Persona = supervisor.nId_Persona
                     join Usuarios u_sup on u_sup.nId_Persona = p_sup.nId_Persona
            where u_sup.nId_Usuario = @id_usuario
        END

  --RANGO DE FECHAS DE CIERRE DE ASISTENCIA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'fechas_cierre_asistencia') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT car.nId_Cierre     as sCodigo,
                   CONCAT(CONVERT(varchar, car.dFecha_Inicio_Asistencia, 23), '/',
                          CONVERT(varchar, car.dFecha_Fin_Asistencia, 23)) as sDescripcion,
                   'FECHAS_CIERRE_ASISTENCIA'                              as sTipo_Configuracion
            FROM cierreAsistenciaReportes car
            UNION
            (SELECT car.nId_Cierre                                        as sCodigo,
                    CONCAT(CONVERT(varchar, car.dFecha_Inicio_Tardanza, 23), '/',
                         CONVERT(varchar, car.dFecha_Fin_Tardanza, 23)) as sDescripcion,
          'FECHAS_CIERRE_TARDANZA'                              as sTipo_Configuracion
             FROM cierreAsistenciaReportes car)
            UNION
            SELECT
                car.nId_Cierre AS sCodigo,
                CONCAT(
                    FORMAT(car.dFecha_Inicio_Tardanza, 'dd/MM/yyyy'), ' - ',
                    FORMAT(car.dFecha_Fin_Tardanza, 'dd/MM/yyyy')
                ) AS sDescripcion,
                'FECHAS_CIERRE_TARDANZA_SUGGESS' AS sTipo_Configuracion
            FROM
                cierreAsistenciaReportes car
            UNION
            (SELECT car.nId_Cierre                           as sCodigo,
                    CONVERT(varchar, car.dFecha_Periodo, 23) as sDescripcion,
     'FECHAS_CIERRE_PERIODO'                  as sTipo_Configuracion
             FROM cierreAsistenciaReportes car)
             ORDER BY car.nId_Cierre DESC
        END

    --METODOLOGIAS Y CATEGORIAS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'metodologias') = 1
        BEGIN
            INSERT INTO @configuraciones
            select m.nId_Metodologia      as sCodigo,
                   m.sNombre              as sDescripcion,
                   'METODOLOGIAS_ACTIVAS' as sTipo_Configuracion
            from Metodologias m
            where m.nEstado = 1
            union
            (select m.nId_Metodologia        as sCodigo,
                    m.sNombre                as sDescripcion,
                    'METODOLOGIAS_INACTIVAS' as sTipo_Configuracion
             from Metodologias m
             where m.nEstado = 0)
            union
            (select c2.nId_Categoria       as sCodigo,
                    c2.sNombre             as sDescripcion,
                    'CATEGORIAS_INACTIVAS' as sTipo_Configuracion
             from Categorias c2
             where c2.nEstado = 0)
            union
            (select c2.nId_Categoria     as sCodigo,
                    c2.sNombre           as sDescripcion,
                    'CATEGORIAS_ACTIVAS' as sTipo_Configuracion
             from Categorias c2
             where c2.nEstado = 1)
            union
            (select CONCAT(m2.nId_Metodologia, c.nId_Categoria)      as sCodigo,
                    CONCAT(m2.nId_Metodologia, '-', c.nId_Categoria) as sDescripcion,
                    'METODOLOGIAS_CATEGORIAS'                        as sTipo_Configuracion
             from Metodologias_Categorias mc
                      join Categorias c on mc.nId_Categoria = c.nId_Categoria
                      join Metodologias m2 on m2.nId_Metodologia = mc.nId_Metodologia)
END

    -- CATEGORIAS POR REQUERIMIENTO
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'categorias') = 1
        BEGIN
            print 'Entro categorias'
INSERT INTO @configuraciones
            select CONVERT(nvarchar(max), c.nId_Categoria) as sCodigo,
                   c.sDescripcion     as sDescripcion,
                   'CATEGORIAS'                            as sTipo_Configuracion
            from Requerimientos_Categoria rc
                     join Categorias c on c.nId_Categoria = rc.nId_Categoria
where rc.nId_Requerimiento = @id_requerimiento
              AND c.nEstado = 1
 END

    -- CATEGORIAS SUGERENCIAS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'categorias_suggest') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT CONVERT(nvarchar(max), c.nId_Categoria) AS sCodigo,
                   c.sDescripcion                          AS sDescripcion,
                   'CATEGORIAS'                            AS sTipo_Configuracion
            FROM Categorias c
            WHERE c.nEstado = 1
        END

    -- CLIENTES EMPRESA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'clientes_suggest') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT c.nId_Cliente      AS sCodigo,
                   p.sPersona_Nombre  AS sDescripcion,
                   'CLIENTES_EMPRESA' AS sTipo_Configuracion
            FROM Clientes c
                     JOIN Personas p ON p.nId_Persona = c.nId_Persona
        END

    -- INFORMACIÓN DE TIPOS DE SOLICITUDES PARA CREAR SOLICITUDES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'categorias_solicitudes') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT cp.nId_Categoria         AS sCodigo,
                   cp.sNombre               AS sDescripcion,
                   'CATEGORIAS_SOLICITUDES' AS sTipo_Configuracion
            FROM Categorias_Permisos cp
            WHERE cp.nEstado = 1
            UNION
            (SELECT ts.nId_Tipo_Solicitud             AS sCodigo,
                    CONCAT(ts.sDescripcion,
                           '-', ts.nId_Categoria,
                           '-', ts.nTipo_Frecuencia,
                           '-', ts.nRangoMin,
                           '-', ts.nRangoMax,
                           '-', ts.nRangoMinAdm,
                           '-', ts.nRangoMaxAdm,
                           '-', ts.nArchivoObligatorio,
                           '-', ts.nAfecta_Asistencia,
                           '-', ts.nMostrar_Calendario,
                           '-', ts.nTiempoAnticipacion,
                  '-', ts.nDiasCalendario,
           '-', ts.nMostrar_Selector) AS sDescripcion,
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS sTipo_Configuracion
             FROM Tipos_Solicitudes ts
             WHERE ts.nEstado = 1
      --and ts.nId_Tipo_Solicitud NOT IN (26)
               and ts.nId_Categoria NOT IN (2))
            UNION
            (select ts.nId_Tipo_Solicitud             AS sCodigo,
                    CONCAT(ts.sDescripcion,
  '-', ts.nId_Categoria,
      '-', ts.nTipo_Frecuencia,
                           '-', ts.nRangoMin,
      '-',
                           CASE
            WHEN ts.nId_Tipo_Solicitud in (3, 10) THEN ROUND(
                          svc.dDias_Vencidos_Laborales + svc.dDias_Ganados_Laborales, 0, 0)
                               WHEN ts.nId_Tipo_Solicitud in (22) THEN (ROUND(svc.dDias_Generados_Laborales, 0, -1))
                             END,
                           '-', ts.nRangoMinAdm,
  '-',
       CASE
  WHEN ts.nId_Tipo_Solicitud in (3, 10) THEN ROUND(
                               svc.dDias_Vencidos_Laborales + svc.dDias_Ganados_Laborales, 0, 0)
                           WHEN ts.nId_Tipo_Solicitud in (22) THEN (ROUND(svc.dDias_Generados_Laborales, 0, -1))
                          END,
     '-', ts.nArchivoObligatorio,
               '-', ts.nAfecta_Asistencia,
                           '-', ts.nMostrar_Calendario,
                        '-', ts.nTiempoAnticipacion,
                           '-', ts.nDiasCalendario,
                           '-', ts.nMostrar_Selector) AS sDescripcion,
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS sTipo_Configuracion
   from Tipos_Solicitudes ts
                      join saldos_Vacaciones_Colaboradores svc
     on svc.nId_Colaborador = @Id_Colaborador
             where ts.nEstado = 1
               and ts.nId_Categoria IN (2))
        END

	 -- INFORMACIÓN DE TIPOS DE SOLICITUDES PARA CREAR SOLICITUDES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'categorias_solicitudess') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT cp.nId_Categoria         AS sCodigo,
                   cp.sNombre               AS sDescripcion,
                   'CATEGORIAS_SOLICITUDES' AS sTipo_Configuracion
            FROM Categorias_Permisos cp
            WHERE cp.nEstado = 1
            UNION
            (SELECT ts.nId_Tipo_Solicitud             AS sCodigo,
                    CONCAT(ts.sDescripcion,
                           '-', ts.nId_Categoria,
                           '-', ts.nTipo_Frecuencia,
                           '-', ts.nRangoMin,
                           '-', ts.nRangoMax,
                           '-', ts.nRangoMinAdm,
                           '-', ts.nRangoMaxAdm,
                           '-', ts.nArchivoObligatorio,
                           '-', ts.nAfecta_Asistencia,
                           '-', ts.nMostrar_Calendario,
                           '-', ts.nTiempoAnticipacion,
                  '-', ts.nDiasCalendario,
           '-', ts.nMostrar_Selector) AS sDescripcion,
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS sTipo_Configuracion
             FROM Tipos_Solicitudes ts
             WHERE ts.nEstado = 1
			and ts.nId_Tipo_Solicitud NOT IN (26)
               and ts.nId_Categoria NOT IN (2))
            UNION
            (select ts.nId_Tipo_Solicitud             AS sCodigo,
                    CONCAT(ts.sDescripcion,
       '-', ts.nId_Categoria,
                           '-', ts.nTipo_Frecuencia,
                           '-', ts.nRangoMin,
                           '-',
                           CASE
            WHEN ts.nId_Tipo_Solicitud in (3, 10) THEN ROUND(
                                       svc.dDias_Vencidos_Laborales + svc.dDias_Ganados_Laborales, 0, 0)
                               WHEN ts.nId_Tipo_Solicitud in (22) THEN (ROUND(svc.dDias_Generados_Laborales, 0, -1))
                             END,
                           '-', ts.nRangoMinAdm,
  '-',
       CASE
  WHEN ts.nId_Tipo_Solicitud in (3, 10) THEN ROUND(
                               svc.dDias_Vencidos_Laborales + svc.dDias_Ganados_Laborales, 0, 0)
                           WHEN ts.nId_Tipo_Solicitud in (22) THEN (ROUND(svc.dDias_Generados_Laborales, 0, -1))
                          END,
     '-', ts.nArchivoObligatorio,
               '-', ts.nAfecta_Asistencia,
                           '-', ts.nMostrar_Calendario,
                        '-', ts.nTiempoAnticipacion,
                           '-', ts.nDiasCalendario,
                           '-', ts.nMostrar_Selector) AS sDescripcion,
                    'CATEGORIA_SOLICITUDES_DETALLE'   AS sTipo_Configuracion
   from Tipos_Solicitudes ts
                      join saldos_Vacaciones_Colaboradores svc
                           on svc.nId_Colaborador = @Id_Colaborador
             where ts.nEstado = 1
               and ts.nId_Categoria IN (2))
        END

    --- REVISION ACTIVA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'revision') = 1
        BEGIN
            INSERT INTO @configuraciones
            select (select sFiltros
                    from Revisiones
                    where nUsuario_Creador = @id_usuario
                      and nEstado = 1) sCodigo,
                   'Reivision Activa'  sDescripcion,
                   'REVISION'          sTipo_Configuracion

        END

    --- ULTIMO CIERRE
    /*IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'ultimo_cierre') = 1
        BEGIN

            Declare @DiasDescuento int = (select TOP 1 CONVERT (int,sCodigo) from Configs c where sTabla = 'DIAS_REVISION')
             Declare @fechaIni date = (SELECT top 1 convert(date,dFecha_Fin) from cierreAsistenciaReportes car order by car.nId_Cierre DESC)
             Declare @fechaFin date = (SELECT dateadd(day,-@DiasDescuento,convert (date,dbo.function_fecha_actual())))

            INSERT INTO @configuraciones
            SELECT
            '1' AS sCodigo,
                CONCAT(@fechaIni,'_', @fechaFin) AS sDescripcion,
                'ULTIMO_CIERRE' AS sTipo_Configuracion
        END*/

    -- SOLICITUD DE USUARIO POR COLABORADOR
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'solicitudes_usuario') = 1
        BEGIN
            INSERT INTO @configuraciones
            select s.nId_Solictud                                                   as sCodigo,
                   concat(ts.nTipo_Frecuencia, '*', ts.nId_Categoria, '*', s.nId_Tipo_Solicitud, '*', s.dFecha_Inicio,
            '*', s.dFecha_Fin, '*', s.dHora_Inicio, '*', s.dHora_Fin) as sDescripcion,
                   'SOLICITUDES_USUARIO' as sTipo_Configuracion
            from Solicitudes s
                     join
                 Tipos_Solicitudes ts on s.nId_Tipo_Solicitud = ts.nId_Tipo_Solicitud
            where s.nId_Colaborador = @Id_Colaborador
              and ts.nId_Categoria not in (4)
              and s.nEstado_Solicitud in (1, 3)
        END

    --PREFIJO PARA PROYECTOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'prefijo_proyecto') = 1
        BEGIN
            INSERT INTO @configuraciones
            select c.sCodigo      as sCodigo,
                   c.sDescripcion as sDescripcion,
          c.sTabla       as sTipo_Configuracion
            from Configs c
            where c.sTabla = 'PREFIJO_PROYECTO'
        END

 --PREFIJO PARA REQUERIMIENTOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'prefijo_requerimiento') = 1
        BEGIN
   INSERT INTO @configuraciones
            select c.sCodigo      as sCodigo,
   c.sDescripcion as sDescripcion,
     c.sTabla       as sTipo_Configuracion
        from Configs c
            where c.sTabla = 'PREFIJO_REQUERIMIENTO'
       AND sCodigo = '1'
        END
    ----TIENE SOLICITUD POR DIA(VACIONES O DIAS) O POR MES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'solicitud_por_dias_mes') = 1
        BEGIN

            DECLARE @FechaActual date = (select CONVERT(date, dbo.function_fecha_actual()))

   INSERT INTO @configuraciones
            SELECT CASE
                      WHEN COUNT(s.nId_Solictud) > 0 THEN CONVERT(nvarchar, 1)
                       ELSE CONVERT(nvarchar, 0)
                       END             sCodigo,
                   'SOLICTID_POR_DIAS' sDescripcion,
                   'SOLICTID_POR_DIAS' sDescripcion

            FROM Solicitudes s
                     JOIN Tipos_Solicitudes ts
         ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud
            WHERE s.nId_Colaborador = @Id_Colaborador
              AND ts.nAfecta_Asistencia = 1
            AND (nTipo_Frecuencia = 2
                OR nTipo_Frecuencia = 3)
              and s.nEstado_Solicitud = 3
              AND @FechaActual BETWEEN s.dFecha_Inicio AND s.dFecha_Fin

        END

    --EFECTOS PARA BANNER CELEBRACIONES
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'efectos_banner') = 1
    BEGIN
        INSERT INTO @configuraciones
        SELECT NULL AS sCodigo,
               'Selecciona un efecto' AS sDescripcion,
               'EFECTOS_BANNER' AS sTipo_Configuracion
        UNION
        SELECT sCodigo AS sCodigo,
               sDescripcion AS sDescripcion,
               'EFECTOS_BANNER' AS sTipo_Configuracion
        FROM Configs
        WHERE sTabla = 'EFECTOS_BANNER'
          AND nEstado = 1


    END

    --PROYECTOS-LIDERES /*By Juan and Eduard*/
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'proyectos_lider') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT CONVERT(nvarchar(max), pl.nId_Lider)                                 as sCodigo,
                   CONCAT(CONVERT(nvarchar(max), pl.nPrincipal), ' ' + sPersona_Nombre) as sDescripcion,
                   'PRY_LIDERES'                                                        as sTipo_Configuracion
            FROM Proyecto_Lider pl
                     INNER JOIN personas_colaboradores pc ON pc.nid_colaborador = pl.nId_Lider
            WHERE pl.nEstado = 1
              AND pl.nId_Proyecto = @id_proyecto
        END

    --COORDINADOR-PROYECTO-REQUERIMIENTO (son coordinadores pertenecientes a un usuario supervisor)
    IF (select count(sConfiguration)
        from @configurations_request
        where sConfiguration = 'coordinador_proyecto_requerimiento') = 1
        BEGIN
            INSERT INTO @configuraciones
            select '0'                                                                        as sCodigo,
                   CONCAT(cp.nId_Coordinador, '-', cp.nId_Proyecto, '-', r.nId_Requerimiento) as sDescripcion,
                'COORDINADOR_PROYECTO_REQUERIMIENTO'                                       as sTipo_Configuracion
            from Requerimientos r
                     join Coordinadores_Proyectos cp on cp.nId_Proyecto = r.nId_Proyecto

                     join Supervisor_Proyecto sp on sp.nId_Proyecto = cp.nId_Proyecto
                     join Coordinadores supervisor on supervisor.nId_Coordinador = sp.nId_Supervisor
                     join Personas p_sup on p_sup.nId_Persona = supervisor.nId_Persona
                     join Usuarios u_sup on u_sup.nId_Persona = p_sup.nId_Persona
       where u_sup.nId_Usuario = @id_usuario
        END

    --PROYECTO-REQUERIMIENTO (son proyectos pertenecientes a un usuario coordinador)
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'proyecto_requerimiento') = 1
  BEGIN
            INSERT INTO @configuraciones
            select '0'                         as sCodigo,
                   CONCAT(cp.nId_Proyecto, '-', r.nId_Requerimiento) as sDescripcion,
                   'PROYECTO_REQUERIMIENTO'            as sTipo_Configuracion
        from Requerimientos r
                     join Coordinadores_Proyectos cp on cp.nId_Proyecto = r.nId_Proyecto
                     join Coordinadores coo on coo.nId_Coordinador = cp.nId_Coordinador
                     join Personas p_sup on p_sup.nId_Persona = coo.nId_Persona
                     join Usuarios u_sup on u_sup.nId_Persona = p_sup.nId_Persona
            where u_sup.nId_Usuario = @id_usuario
        END

    --SOLO REQUERIMIENTOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'todo_requerimiento') = 1
BEGIN
            INSERT INTO @configuraciones
            select r.nId_Requerimiento     as sCodigo,
                   r.sNombre     AS sDescripcion,
                   'REQUERIMIENTOS_FILTRO' as sTipo_Configuracion
            from Requerimientos r
        END

    -- REQUERIMIENTOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'requerimiento_supervisor') = 1
        BEGIN
           INSERT INTO @configuraciones
            select r.nId_Requerimiento         as sCodigo,
                   r.sNombre     AS sDescripcion,
                   'REQUERIMIENTOS_SUPERVISOR' as sTipo_Configuracion
            from Requerimientos r
                     JOIN Proyectos pry ON pry.nId_Proyecto = r.nId_Proyecto
                     JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = pry.nId_Proyecto
                -- join Coordinadores_Proyectos cp on cp.nId_Coordinador = c.nId_Coordinador
                -- JOIN Personas p ON c.nId_Persona = p.nId_Persona
                     join Supervisor_Proyecto sp on sp.nId_Proyecto = cp.nId_Proyecto
                     join Coordinadores supervisor on supervisor.nId_Coordinador = sp.nId_Supervisor
                     join Personas p_sup on p_sup.nId_Persona = supervisor.nId_Persona
                     join Usuarios u_sup on u_sup.nId_Persona = p_sup.nId_Persona
            WHERE u_sup.nId_Usuario = @id_usuario
              AND pry.nEstado = 1;
        END

    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'requerimiento_coordinador') =
       1
        BEGIN
 INSERT INTO @configuraciones
            select r.nId_Requerimiento          as sCodigo,
                   r.sNombre     AS sDescripcion,
                   'REQUERIMIENTOS_COORDINADOR' as sTipo_Configuracion
            from Requerimientos r
                     JOIN Proyectos pry ON pry.nId_Proyecto = r.nId_Proyecto
                     JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = pry.nId_Proyecto
                -- join Coordinadores_Proyectos cp on cp.nId_Coordinador = c.nId_Coordinador
                -- JOIN Personas p ON c.nId_Persona = p.nId_Persona
                -- join Supervisor_Proyecto sp on sp.nId_Proyecto = cp.nId_Proyecto
                     join Coordinadores coor on coor.nId_Coordinador = cp.nId_Coordinador
                     join Personas p_coor on p_coor.nId_Persona = coor.nId_Persona
               join Usuarios u_coor on u_coor.nId_Persona = p_coor.nId_Persona
            WHERE u_coor.nId_Usuario = @id_usuario
              AND pry.nEstado = 1;
 END
    -- IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'todo_requerimiento') = 1
    --  BEGIN
    --   INSERT INTO @configuraciones
    --   select
    --    r.nId_Requerimiento as sCodigo,
    --    r.sNombre as sDescripcion,
  --    'REQUERIMIENTOS_FILTRO' as sTipo_Configuracion
    --   from
    --    Requerimientos r
    -- END

    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'requerimientos_lider') = 1
        BEGIN
   IF EXISTS (SELECT *
              FROM Proyecto_Lider pl
                       WHERE pl.nId_Lider = @id_colaborador
                         AND pl.nPrincipal = 1
                 AND pl.nEstado = 1
AND pl.nId_Proyecto = @id_proyecto)
                BEGIN
         INSERT INTO @configuraciones
     SELECT r.nId_Requerimiento    as sCodigo,
                      r.sNombre     AS sDescripcion,
             'REQUERIMIENTOS_LIDER' as sTipo_Configuracion
                    FROM Requerimientos r
               WHERE r.nId_Proyecto = @id_proyecto and r.nEstado = 1 -- acá
                END
 ELSE
                BEGIN
                    INSERT INTO @configuraciones
               SELECT r.nId_Requerimiento    as sCodigo,
                          r.sNombre     AS sDescripcion,
                           'REQUERIMIENTOS_LIDER' as sTipo_Configuracion
                    FROM Requerimientos r
                    WHERE r.nId_Lider = @id_colaborador
                      AND r.nId_Proyecto = @id_proyecto and r.nEstado = 1 -- acá
                END
        END

    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'dias_laborales') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT '1'                       as sCodigo,
                   STRING_AGG(hd.nDia, ', ') AS sDescripcion,
                   'DIAS_LABORALES'          as sTipo_Configuracion
            FROM Horarios_Colaboradores hc
                     JOIN Horarios_Detalles hd ON hd.nId_Horario = hc.nId_Horario
                     JOIN personas_colaboradores pc ON pc.nid_colaborador = hc.nId_Colaborador
                     JOIN Usuarios u ON u.nId_Persona = pc.nid_persona
            WHERE u.nId_Usuario = @id_usuario
        END

    --RUBROS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'rubros') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT r.nId_Rubro    AS sCodigo,
                   r.sDescripcion AS sDescripcion,
   'RUBRO'        AS sTipo_Configuracion
            FROM Rubros r
            WHERE r.nEstado = 1;
        END

   -- EMPRESAS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'empresas_suggest') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT e.nId_Empresa     AS sCodigo,
                   e.sNombre_Empresa AS sDescripcion,
                   'EMPRESAS'        AS sTipo_Configuracion
            FROM Empresas e
        END

    -- MONEDAS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'monedas') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT c.sCodigo                             AS sCodigo,
                   CONCAT(c.sIcono, '-', c.sDescripcion) AS sDescripcion,
                   'MONEDAS'      AS sTipo_Configuracion
            FROM Configs c
   WHERE sTabla = 'TIPO_MONEDA'
       AND nEstado = 1
     END

    -- METODOS DE PAGO
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'metodos_pago') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT c.sCodigo      AS sCodigo,
                  c.sDescripcion AS sDescripcion,
                   'METODO_PAGO'  AS sTipo_Configuracion
            FROM Configs c
            WHERE sTabla = 'METODO_PAGO'
      AND c.nEstado = 1
        END

    -- ESTADOS DE SOLICITUD TESORERIA
    IF (select count(sConfiguration)
        from @configurations_request
    where sConfiguration = 'estados solicitud tesoreria') = 1
        BEGIN
            INSERT INTO @configuraciones
 SELECT c.sCodigo                    AS sCodigo,
                   c.sDescripcion               AS sDescripcion,
                   'ESTADO_SOLICITUD_TESORERIA' AS sTipo_Configuracion
            FROM Configs c
            WHERE sTabla = 'ESTADO_SOLICITUD_TESORERIA'
              AND c.nEstado = 1
       END

    -- EMPRESA
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'client_company_suggest') = 1
        BEGIN
  INSERT INTO @configuraciones
            select nId_Cliente     as sCodigo,
     sNombre_Cliente as sDescripcion,
                   'COMPANY'       as sTipo_Configuracion
            from v_Listado_Clientes_MG
        END

    --CORREOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'emails_list') = 1
        BEGIN
            INSERT INTO @configuraciones
            select nId_Usuario as sCodigo,
                   sEmail      as sDescripcion,
                   'CORREO'    as sTipo_Configuracion
            from Usuarios
            WHERE nEstado = 1
        END

    -- SOLICITUDES PENDIENTES DEL COLABORADOR
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'solicitudes_pendientes') = 1
        BEGIN
            INSERT INTO @configuraciones
            select count(s.nId_Solictud)                as sCodigo,
                   null                                 as sDescripcion,
                   'SOLICITUDES_PENDIENTES_COLABORADOR' as sTipo_Configuracion
            -- add a column to get the count of the pending requests

            from Solicitudes s
            where s.nId_Colaborador = @Id_Colaborador
              and s.nEstado_Solicitud = 1
        END


    -- TOTAL PROYECTOS
    IF (SELECT COUNT(sConfiguration) from @configurations_request WHERE sConfiguration = 'proyectos_total') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT COUNT(p.nId_Proyecto) as sCodigo,
                   null                  as sDescripcion,
                   'PROYECTOS_TOTAL'     as sTipo_Configuracion
            FROM Proyectos p
        END

    --TIPOS DE ACCESO PROYECTOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'acceso_proyectos') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT c.sCodigo              as sCodigo,
                   c.sDescripcion         as sDescripcion,
                   'TIPO_ACCESO_PROYECTO' as sTipo_Configuracion
            FROM Configs c
            WHERE c.sTabla = 'TIPO_ACCESO_PROYECTO'
     ORDER BY nOrdenamiento ASC
        END
    --PLANTILLAS
    --TIPOS DE ACCESO PROYECTOS
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'plantillas_contratos') = 1
        BEGIN
            INSERT INTO @configuraciones
            SELECT CONCAT(ct.nEmpresa_Usuaria, '-', ct.sTipo_Contratacion) as sCodigo,
            ct.sId_DriveFile                           as sDescripcion,
                   'TEMPLATES'                                             as sTipo_Configuracion
            FROM Contract_Templates ct
            WHERE nEstado = 1
        END

--PROYECTOS LISTADO RESPONSIVE
    IF (SELECT COUNT(sConfiguration) FROM @configurations_request WHERE sConfiguration = 'proyectos_listado') = 1
        BEGIN
            IF @slug_listado = 'TODOS'
                BEGIN
                    INSERT INTO @configuraciones
                    SELECT p.nId_Proyecto AS sCodigo,
                           p.sNombre AS sDescripcion,
                           'PROYECTOS_LISTADO' AS sTipo_Configuracion
                    FROM Proyectos p
                END
            ELSE
                IF @slug_listado = 'EQUIPO'
                    BEGIN
INSERT INTO @configuraciones
                        SELECT DISTINCT p.nId_Proyecto      AS sCodigo,
                                        p.sNombre           AS sDescripcion,
                                        'PROYECTOS_LISTADO' AS sTipo_Configuracion
                        FROM Proyectos p
                                 JOIN Colaboradores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto
                                 JOIN Team_Colaboradors tc ON tc.nId_Colaborador = cp.nId_Colaborador
                                 JOIN Team t ON t.nId_Team = tc.nId_Team
                      JOIN personas_colaboradores pc ON pc.nid_colaborador = t.nId_Lider
                   JOIN Usuarios u ON u.nId_Persona = pc.nid_persona
                   WHERE u.nId_Usuario = @id_usuario
                    END
        END

    --CIERRE TAREO
   IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'cierre-tareo') = 1
        BEGIN
         INSERT INTO @configuraciones
   SELECT TOP 1
   nId_Cierre AS sCodigo,
   dFecha_Cierre AS sDescripcion,
   'CIERRE_TAREO' as sTipo_Configuracion
   FROM Cierre_Horas where nEstado_Cierre = 1
   AND nEstado = 1
   ORDER BY nId_Cierre DESC
        END

     --CIERRE TAREO PENDIENTES
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'cierre-tareo-pendientes') = 1
        BEGIN
         INSERT INTO @configuraciones
   SELECT
       nId_Cierre AS sCodigo,
       CONCAT(MONTH(dFecha_Cierre), '-', YEAR(dFecha_Cierre) ) AS sDescripcion,
       'CIERRE_TAREO_PENDIENTES' as sTipo_Configuracion
   FROM
       Cierre_Horas
   WHERE
       nEstado_Cierre = 0
       AND nEstado = 1
       AND dFecha_Cierre <= EOMONTH(DATEADD(MONTH, 1, GETDATE()))
   ORDER BY
       nId_Cierre ASC;
        END

    --CATEGORIAS TODO
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'categorias_all') = 1
        BEGIN
         INSERT INTO @configuraciones
   SELECT
       nId_Categoria AS sCodigo,
       sNombre AS sDescripcion,
       'CATEGORIAS_ALL' as sTipo_Configuracion
   FROM
       Categorias
   WHERE
       nEstado = 1
        END

        --APROBADORES
     IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'aprobadores') = 1
        BEGIN
     INSERT INTO @configuraciones
        SELECT DISTINCT
       nId_Aprobador AS sCodigo,
       sNombre_Aprobador AS sDescripcion,
       'APROBADORES_SOLICITUDES' AS sTipo_Configuracion
      FROM v_Listado_Aprobadores_Solicitudes
      ORDER BY nId_Aprobador
     END

                        --CODIGOS EMAIL PARA MAILING
     IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'emails_personal_groups') = 1
        BEGIN
         INSERT INTO @configuraciones
      SELECT  *
      from v_listado_emails_personas
     END

  --CODIGO PARA REMITENTE
  IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'remitente') = 1
        BEGIN
         INSERT INTO @configuraciones
      SELECT  DISTINCT
    nId_Usuario AS sCodigo,
    sEmail AS sDescripcion,
    'REMITENTE' AS sTipo_Configuracion
      from usuarios

  UNION
   SELECT  DISTINCT
   sCodigo AS sCodigo,
    sDescripcion AS sDescripcion,
    'REMITENTE' AS sTipo_Configuracion
     from Configs WHERE nId_Config = 328
 END

             --LOCALIZACION --TODO: CAMBIAR SEGÚN SEDE
     IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'localizacion_predeterminada') = 1
        BEGIN
         INSERT INTO @configuraciones
   SELECT
   0 AS sCodigo,
   (SELECT
   sValor_Minimo AS latitude,
   sValor_Maximo AS longitude,
   0 AS accuracy
   FROM
   Configs
   WHERE sTabla  = 'LOCALIZACION'
   FOR JSON PATH
   ) AS sDescripcion,
   'LOCALIZACION_PREDETERMINADA' AS sTipo_Configuracion
     END

     --MODALIDAD DEL COLABORADOR POR ASISTENCIA
     IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'modalidad_asistencia') = 1
        BEGIN
         DECLARE @date DATETIME = CONVERT(DATETIME, @slug_listado)
   DECLARE @modalidad INT = dbo.fn_get_modalidad_colaborador(@Id_Colaborador, @date)

         INSERT INTO @configuraciones
   SELECT
   @modalidad AS sCodigo,
   sDescripcion,
   'MODALIDAD_ASISTENCIA' AS sTipo_Configuracion
   FROM Configs
   WHERE sTabla = 'TIPO_MODALIDAD'
   AND sCodigo = CONVERT(NVARCHAR, @modalidad)
     END

 -- listado de creadores de emails (módulo de comunicaciones)
    IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'mailing_creator') = 1
        BEGIN
   INSERT INTO @configuraciones
   select
    distinct
    mt.nUsuario_Creador as sCodigo,
    concat_ws(' ' ,p.sPrimer_Nombre, p.sApe_Paterno)    as sDescripcion,
    'MAILING_CREATOR'   as sTipo_Configuracion
   from MailingTemplate mt
   inner join usuarios s on s.nId_usuario = mt.nUsuario_Creador
   inner join Personas p on p.nId_persona = s.nId_persona
        END


 -- listado de mailing (nombres de correos):
 IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'mailing_list') = 1
    BEGIN
        INSERT INTO @configuraciones
        select
   nId_MailingTemplate sCodigo,
   sName sDescripcion,
   'MAILING_LIST' sTipo_Configuracion
  from MailingTemplate
    END

     -- diferencia de distancia maxima para marcar por botonera
 IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'rango_marcas') = 1
    BEGIN
        INSERT INTO @configuraciones
       select
   sCodigo,
   sDescripcion,
   'RANGO_MARCAS' sTipo_Configuracion
  from Configs c
  WHERE sTabla = 'RANGO_MARCAS'
    END

 IF (SELECT COUNT(sConfiguration) from @configurations_request where sConfiguration = 'evaluation_template') = 1
	BEGIN
		INSERT INTO @configuraciones
		SELECT
		nId_Form_Template sCodigo,
		sName sDecripcion,
		'EVALUATION_TEMPLATE' sTipo_Configuration
		FROM Form_Template
	END
	
IF (SELECT COUNT(sConfiguration) from @configurations_request where sConfiguration = 'evaluation') = 1
	BEGIN
		INSERT INTO @configuraciones
		SELECT
		nId_Evaluation sCodigo,
		sName sDecripcion,
		'EVALUATION' sTipo_Configuration
		FROM Evaluations
	END

    --RETURN CONFIGURACIONES
    SELECT * FROM @configuraciones --order by sDescripcion
END;
GO
```