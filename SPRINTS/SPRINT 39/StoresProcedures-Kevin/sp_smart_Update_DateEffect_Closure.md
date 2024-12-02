```sql
CREATE PROCEDURE sp_smart_Update_DateEffect_Closure

@nCantidadDias INT = NULL,

@dSinceEffectDay DATE = NULL,

@nUsuario_Update INT = NULL

AS

BEGIN

SET

NOCOUNT ON;

BEGIN

TRY

DECLARE @UltimaFecha DATE;

  

DECLARE @ExisteConfiguración BIT = 0;

  

DECLARE @MensajeError VARCHAR(500);

  

DECLARE @IdProximoCierre INT;

  

DECLARE @FechaActual DATE = CAST(GETDATE() AS DATE);

  

DECLARE @DiasVigentes INT;

  

SELECT

@DiasVigentes = nCantidadDias

FROM

ClosingDate

WHERE

nEstadoDate = 1;

  

IF @nCantidadDias <= 0

BEGIN

SET

@MensajeError = 'La cantidad de días debe ser mayor a 0';

  

THROW 51000,

@MensajeError,

1;

END

IF @dSinceEffectDay < @FechaActual

BEGIN

SET

@MensajeError = 'La fecha de efecto no puede ser menor a la fecha actual';

  

THROW 51001,

@MensajeError,

1;

END

IF @DiasVigentes IS NOT NULL

AND @DiasVigentes = @nCantidadDias

BEGIN

SET

@MensajeError = 'La cantidad de días debe ser diferente a la configuración vigente';

  

THROW 51002,

@MensajeError,

1;

END

SELECT

TOP 1

@UltimaFecha = dSinceEffectDay,

@ExisteConfiguración = 1,

@IdProximoCierre = nIdClosingDate

FROM

ClosingDate

WHERE

nEstadoDate = 2

-- Solo registros pendientes

AND dSinceEffectDay >= @FechaActual

ORDER BY

dSinceEffectDay ASC;

  

BEGIN TRANSACTION;

  

IF @dSinceEffectDay = @FechaActual

BEGIN

UPDATE

ClosingDate

SET

nEstadoDate = 0,

dDatetime_Update = GETDATE(),

nUsuario_Update = @nUsuario_Update

WHERE

nEstadoDate = 1;

  

IF @ExisteConfiguración = 1

BEGIN

UPDATE

ClosingDate

SET

nEstadoDate = 0,

dDatetime_Update = GETDATE(),

nUsuario_Update = @nUsuario_Update

WHERE

nIdClosingDate = @IdProximoCierre

AND nEstadoDate = 2;

END

-- Insertar nuevo registro como activo

INSERT

INTO

ClosingDate (

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

1,

-- Activo

GETDATE()

);

END

ELSE

-- Si es una fecha futura

BEGIN

-- Si existe una fecha pendiente, desactivarla

IF @ExisteConfiguración = 1

BEGIN

UPDATE

ClosingDate

SET

nEstadoDate = 0,

dDatetime_Update = GETDATE(),

nUsuario_Update = @nUsuario_Update

WHERE

nIdClosingDate = @IdProximoCierre

AND nEstadoDate = 2;

END

-- Insertar nuevo registro como pendiente

INSERT

INTO

ClosingDate (

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

2,

-- Pendiente

GETDATE()

);

END

  

COMMIT TRANSACTION;

  

SELECT

'Configuración actualizada exitosamente' as Mensaje,

1 as Success,

SCOPE_IDENTITY() as nIdClosingDate,

@nCantidadDias as nCantidadDias,

@dSinceEffectDay as dSinceEffectDay,

CASE

WHEN @dSinceEffectDay = @FechaActual THEN 1

ELSE 2

END as nEstadoDate;

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