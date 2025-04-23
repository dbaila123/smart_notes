```sql
create table Total_Min_Apro_Pend (
	nId_Colaborador int, 
	sNombre_Colaborador nvarchar(max), 
	dTardanza_Con_Tolerancia int, 
	nTotal_Min_No_Recuperables int,
    nTotal_Min_Contra_Per_Apro int,
    nTotal_Min_Favor_CompRecu_Apro int,
    nTotal_Min_Contra_Per_Pend int,
    nTotal_Min_Favor_CompRecu_Pend int,
    nTotal_Min_Favor_Extra_Apro int,
    nTotal_Min_Favor_Extra_Pend int,
    nTotal_Min_Favor_Extra_AP int,
    nTotal_Min_Favor_CompRecu_AP int,
    nTotal_Min_Contra_Per_AP int,
    nTotal_Bolsa_Cierre int,
    nTotal_Contra int,
    nTotal_Favor int,
    nTotal_Minutos_Recuperar int
);

CREATE INDEX IX_Total_Min_Apro_Pend_Colaborador
ON Total_Min_Apro_Pend (nId_Colaborador);
```
