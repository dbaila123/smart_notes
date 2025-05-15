```sql
CREATE PROCEDURE [dbo].[sp_smart_evaluation_get_evaluators_assessed_template](

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

  

IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 1)

OR NOT EXISTS(SELECT 1 FROM @aFormType)

BEGIN

INSERT INTO #Evaluators (nId_Evaluator, sEvaluator_Name, nId_Assessed, sAssessed_Name, nId_FormType)

SELECT

c.nId_Colaborador AS nId_Evaluator,

p1.sPersona_Nombre AS Evaluator_Name,

cs.nId_Supervisor AS nId_Assessed,

p2.sPersona_Nombre AS Assessed_Name,

1

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

  

  

IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 2)

OR NOT EXISTS(SELECT 1 FROM @aFormType)

BEGIN

INSERT INTO #Evaluators

SELECT

c.nId_Colaborador as nId_Evaluator,

pc.sPersona_Nombre as Evaluator_Name,

cs.nId_Colaborador as nId_Assessed,

pcs.sPersona_Nombre as Assessed_Name,

2 as nId_FormType

FROM Colaboradores c

JOIN Personas pc on c.nId_Persona = pc.nId_Persona

JOIN colaboradores_supervisor cs

ON c.nId_Colaborador = cs.nId_Supervisor

AND cs.nEstado = 1

AND cs.bPrincipal = 1

JOIN Colaboradores ccs on cs.nId_Colaborador = ccs.nId_Colaborador

JOIN Personas pcs on ccs.nId_Persona = pcs.nId_Persona

WHERE (

NOT EXISTS (SELECT 1 FROM @aColaboradores)

OR c.nId_Colaborador IN (SELECT nId_Colaborador FROM @aColaboradores)

)

AND c.nEstado_Colaborador = 1

AND ccs.nEstado_Colaborador = 1

END

  

IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 4)

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

4 as nId_FormType

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

  

IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 5)

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

---------------------------------------

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

------------------

INSERT INTO #Evaluators

SELECT

cl.nId_Evaluator,

cl.Evaluator_Name,

le.nId_Assessed,

le.Assessed_Name,

5 as nId_FormType

FROM #Colaborador_Lideres cl

JOIN #Lideres_Equipos le ON cl.nId_Assessed = le.nId_Evaluator

WHERE

cl.nId_Evaluator != le.nId_Assessed

and cl.nId_Evaluator not in (select nId_Evaluator

from #Lideres_Equipos)

  

END

  

SELECT *

FROM #Evaluators

ORDER BY nId_Evaluator, nId_FormType

END
```