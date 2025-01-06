```SQL
CREATE PROCEDURE sp_Requeriments_Update

@nId_Requerimiento int,

@nId_Proyecto int,

@sNombre nvarchar(MAX),

@nId_Lider int = 0,

@sDescripcion nvarchar(MAX),

@nId_Metodologia int,

@dFecha_Inicio date,

@dFecha_Fin date,

@nUsuario_Update int,

@dDatetime_Update datetime,

@sCodigo nvarchar(MAX) = NULL

AS

BEGIN

DECLARE @UpdatedId INT;

  

BEGIN TRY

UPDATE REQUERIMIENTOS

SET nId_Proyecto = @nId_Proyecto,

sNombre = @sNombre,

nId_Lider = @nId_Lider,

sDescripcion = @sDescripcion,

nId_Metodologia = @nId_Metodologia,

dFecha_Inicio = @dFecha_Inicio,

dFecha_Fin = @dFecha_Fin,

nUsuario_Update = @nUsuario_Update,

dDatetime_Update = @dDatetime_Update,

sCodigo = ISNULL(@sCodigo, sCodigo)

WHERE nId_Requerimiento = @nId_Requerimiento;

  

SET @UpdatedId = 1;

SELECT @UpdatedId AS Result;

END TRY

BEGIN CATCH

SET @UpdatedId = 0;

SELECT @UpdatedId AS Result;

END CATCH

END;
```