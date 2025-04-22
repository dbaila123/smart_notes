```sql
CREATE PROCEDURE [dbo].[sp_estado_tareo_colaborador_asistencia]

(

@fecha_ini date,

@fecha_fin date,

@id_colaborador int

)

AS

BEGIN

DECLARE @nTotalMinutosRecuperar INT;

  

SELECT @nTotalMinutosRecuperar = nTotal_Minutos_Recuperar

FROM Total_Min_Apro_Pend

WHERE nId_Colaborador = @id_colaborador;

  

SELECT *,

@nTotalMinutosRecuperar AS nTotal_Minutos_Recuperar

FROM v_Listado_Tareo_Estado_Colaboradores

WHERE dDate BETWEEN @fecha_ini and @fecha_fin

AND nId_Colaborador = @id_colaborador;

  

END
```