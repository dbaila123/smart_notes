```sql
CREATE OR ALTER PROCEDURE sp_get_hours_by_collaborator(
	@dFecha DATETIME,    
    @sIds_Colaborador NVARCHAR(MAX) = NULL
)AS
BEGIN

DECLARE @nId_Collaborators TABLE (nId INT)
DECLARE @dFecha_cierre varchar(20)

SELECT TOP 1 @dFecha_cierre = dFecha_cierre from closure_request order by nid_cierre desc

INSERT INTO @nId_Collaborators
SELECT value
FROM STRING_SPLIT(@sIds_Colaborador, ',');
     
Select 
	nId_Colaborador,
	dTardanza_Con_Tolerancia,
	nTotal_Min_No_Recuperables AS dTardanza_Sin_Tolerancia,
	nTotal_Min_Contra_Per_AP AS nPermisos,
	nTotal_Favor AS nAFavor,
	nTotal_Contra AS nEnContra,
	nTotal_Min_Favor_Extra_AP AS nExtra,
	@dFecha_cierre AS dFecha_cierre,
	nTotal_Min_Favor_CompRecu_AP AS dCompensacion_Minutos,
	nTotal_Bolsa_Cierre AS dBolsa_Cierre
From Total_Min_Apro_Pend where nId_Colaborador IN(SELECT nId From @nId_Collaborators)

END
```