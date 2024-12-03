```sql
CREATE PROCEDURE sp_smart_Update_DateEffect_Closure

@nCantidadDias INT = NULL,

@dSinceEffectDay DATE = NULL,

@nUsuario_Update INT = NULL

AS

BEGIN

SET NOCOUNT ON;

BEGIN TRY

DECLARE @UltimaFecha DATE;

DECLARE @ExisteConfiguración BIT = 0;

DECLARE @MensajeError VARCHAR(500);

DECLARE @IdProximoCierre INT;

DECLARE @FechaActual DATE = CAST(GETDATE() AS DATE);

DECLARE @DiasVigentes INT;

  

-- Obtener días vigentes de la fecha de cierre más cercana a la fecha actual

SELECT TOP 1 @DiasVigentes = nCantidadDias

FROM ClosingDate

WHERE nEstadoDate = 1

AND dSinceEffectDay <= @FechaActual

ORDER BY dSinceEffectDay DESC;

  

-- Validaciones

IF @nCantidadDias <= 0

BEGIN

SET @MensajeError = 'La cantidad de días debe ser mayor a 0';

THROW 51000, @MensajeError, 1;

END

  

IF @dSinceEffectDay < @FechaActual

BEGIN

SET @MensajeError = 'La fecha de efecto no puede ser menor a la fecha actual';

THROW 51001, @MensajeError, 1;

END

  

IF @DiasVigentes IS NOT NULL AND @DiasVigentes = @nCantidadDias

BEGIN

SET @MensajeError = 'La cantidad de días debe ser diferente a la configuración vigente más cercana a la fecha actual';

THROW 51002, @MensajeError, 1;

END

  

BEGIN TRANSACTION;

  

-- Inhabilitar registros mayores o iguales a la fecha actual

UPDATE ClosingDate

SET

nEstadoDate = 0,

dDatetime_Update = GETDATE(),

nUsuario_Update = @nUsuario_Update

WHERE dSinceEffectDay >= @FechaActual

AND nEstadoDate = 1;

  

-- Insertar nuevo registro con estado 1

INSERT INTO ClosingDate (

nCantidadDias,

dSinceEffectDay,

nUsuario_Creador,

nEstadoDate,

dDatetime_Creador

)

VALUES (

@nCantidadDias,

@dSinceEffectDay,

@nUsuario_Update,

1, -- Activo

GETDATE()

);

  

COMMIT TRANSACTION;

  

SELECT

'Configuración actualizada exitosamente' as Mensaje,

1 as Success,

SCOPE_IDENTITY() as nIdClosingDate,

@nCantidadDias as nCantidadDias,

@dSinceEffectDay as dSinceEffectDay,

1 as nEstadoDate;

  

END TRY

BEGIN CATCH

IF @@TRANCOUNT > 0

ROLLBACK TRANSACTION;

  

SELECT

ERROR_MESSAGE() as Mensaje,

0 as Success,

0 as nIdClosingDate,

NULL as nCantidadDias,

NULL as dSinceEffectDay,

NULL as nEstadoDate;

END CATCH

END;
```