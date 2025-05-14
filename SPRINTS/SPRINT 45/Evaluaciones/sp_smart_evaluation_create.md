```sql
CREATE PROCEDURE [dbo].[sp_smart_evaluation_create](

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

WHERE ft.nId_Form_Type IS NULL

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

INSERT INTO Forms (sName, sDescription, nId_Evaluation, nUser_Creator, dDateTime_Creator, nId_Form_Type, nId_Form_SubType)

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

END) as nId_Form_SubType

FROM @FormTypeTemplate t

JOIN Form_Type ft on t.nId_FormType = ft.nId_Form_Type

  

--INSERT QUESTIONS

INSERT INTO Questions (nId_Form, sText, nId_Type_Question, bRequired, sPlaceHolder, nId_Competence, nUser_Creator, dDateTime_Creator)

SELECT

formt.nId_Form,

qt.sText,

qt.nId_Type_Question,

qt.bRequired,

qt.sPlaceHolder,

qt.nId_Competence,

@UserCreator,

GETDATE() as dDateTime_Creator

FROM Form_Template ft

JOIN Questions_Template qt ON ft.nId_Form_Template = qt.nId_Form_Template

JOIN @FormTypeTemplate ftt ON ft.nId_Form_Template = ftt.nId_Template

JOIN @FormTable formt ON (

(formt.nId_Form_SubType IS NULL AND ftt.nId_FormType = formt.nId_Form_Type)

OR

(formt.nId_Form_SubType IS NOT NULL AND ftt.nId_FormType = formt.nId_Form_SubType)

) ORDER BY formt.nId_Form

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

  

COMMIT TRANSACTION;

END TRY

BEGIN CATCH

IF @@TRANCOUNT > 0

ROLLBACK TRANSACTION;

  

THROW;

END CATCH

  

END
```