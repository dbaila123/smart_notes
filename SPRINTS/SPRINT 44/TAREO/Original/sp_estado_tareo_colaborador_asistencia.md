```sql
CREATE PROCEDURE [dbo].[sp_estado_tareo_colaborador_asistencia]

(

@fecha_ini date,

@fecha_fin date,

@id_colaborador int

)

AS

BEGIN

SELECT * FROM v_Listado_Tareo_Estado_Colaboradores

WHERE dDate BETWEEN @fecha_ini and @fecha_fin

AND nId_Colaborador = @id_colaborador;

  

END
```