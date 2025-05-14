```sql
CREATE PROCEDURE [dbo].[sp_smart_evaluation_get_evaluators_assessed]

(

@nId_Evaluation INT,

@sFilterOne VARCHAR(MAX) = NULL, --FILTRO EQUIPOS

@pnum INT = NULL, --Número de Páginas

@size INT = NULL --Cantidad de Registros por página

)

AS

BEGIN

DECLARE @script AS nvarchar(max)

DECLARE @whereClause AS nvarchar(max)

DECLARE @paginationClause AS nvarchar(max)

DECLARE @conditions AS TABLE (condition nvarchar(max))

--TEAM FILTER

IF @sFilterOne IS NOT NULL

BEGIN

DECLARE @CollaboratorsListTeam VARCHAR(MAX);

SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)

INSERT INTO @conditions VALUES(N'nId_Evaluator in('+@CollaboratorsListTeam+')')

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

print(@pnum)

print(@size)

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

SELECT *

FROM v_list_evaluators_assessed

WHERE nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation) + @whereClause +'

ORDER BY nId_Evaluator ' +

@paginationClause

  

EXECUTE sp_executesql @script

  

END
```