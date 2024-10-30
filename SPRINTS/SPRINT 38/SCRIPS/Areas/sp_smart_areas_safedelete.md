```sql
CREATE PROCEDURE dbo.sp_smart_areas_safedelete

@nId_Area INT,

@nUsuario_Delete INT,

@dDatetime_Delete DATETIME,

@nUsuario_Update INT,

@dDatetime_Update DATETIME

AS

BEGIN

SET NOCOUNT ON;

  

UPDATE tenant0001.dbo.Areas

SET nEstado = 0,

nUsuario_Delete = @nUsuario_Delete,

dDatetime_Delete = @dDatetime_Delete,

nUsuario_Update = @nUsuario_Update,

dDatetime_Update = @dDatetime_Update

WHERE nId_Area = @nId_Area;

  

END;
```