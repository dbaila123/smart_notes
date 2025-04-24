```sql
CREATE PROCEDURE [dbo].[sp_get_hours_by_collaborator](

@dFecha DATETIME,

@sIds_Colaborador NVARCHAR(MAX) = NULL,

@bPendiente BIT = 1

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

CASE

WHEN @bPendiente = 0 THEN nTotal_Min_Contra_Per_Apro

ELSE nTotal_Min_Contra_Per_AP

END AS nPermisos,

CASE

WHEN @bPendiente = 0 THEN nTotal_Min_Favor_Extra_Apro + nTotal_Min_Favor_CompRecu_Apro + nTotal_Bolsa_Cierre

ELSE nTotal_Min_Favor_Extra_AP + nTotal_Min_Favor_CompRecu_AP + nTotal_Bolsa_Cierre

END AS nAFavor,

CASE

WHEN @bPendiente = 0 THEN nTotal_Min_Contra_Per_Apro + dTardanza_Con_Tolerancia + nTotal_Min_No_Recuperables

ELSE nTotal_Min_Contra_Per_AP + dTardanza_Con_Tolerancia + nTotal_Min_No_Recuperables

END AS nEnContra,

CASE

WHEN @bPendiente = 0 THEN nTotal_Min_Favor_Extra_Apro

ELSE nTotal_Min_Favor_Extra_AP

END AS nExtra,

@dFecha_cierre AS dFecha_cierre,

CASE

WHEN @bPendiente = 0 THEN nTotal_Min_Favor_CompRecu_Apro

ELSE nTotal_Min_Favor_CompRecu_AP

END AS dCompensacion_Minutos,

nTotal_Bolsa_Cierre AS dBolsa_Cierre

From Total_Min_Apro_Pend where nId_Colaborador IN(SELECT nId From @nId_Collaborators)

  

END```