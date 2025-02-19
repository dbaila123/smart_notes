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

@sCodigo nvarchar(MAX) = NULL,

@sCodigo_sinprefijo nvarchar(max),

@sNombre_SinPrefijo nvarchar(max)

AS

BEGIN

DECLARE @UpdatedId INT;

DECLARE @NombreFinal NVARCHAR(MAX);

BEGIN TRY

IF @sCodigo IS NULL

BEGIN

SET @sCodigo = '-';

END

  

IF @sCodigo <> '-'

BEGIN

SET @NombreFinal = CONCAT(@sCodigo, ' ', + @sNombre)

END

ELSE

BEGIN

SET @NombreFinal = CONCAT('RQ', ' - ', + @sNombre)

END

IF EXISTS (

SELECT 1

FROM REQUERIMIENTOS

WHERE sCodigo = @sCodigo

) AND @sCodigo NOT IN ('RQ - ', '-')

BEGIN

SET @UpdatedId = 0;

SELECT @UpdatedId AS Result;

--RETURN;

END

--DECLARE @codigoQ VARCHAR(MAX) = NULL

--set @codigoQ = iif(@sCodigo = 'null', concat('RQ',cast(@nId_Requerimiento as varchar)), @sCodigo)

UPDATE REQUERIMIENTOS

SET nId_Proyecto = @nId_Proyecto,

sNombre = @NombreFinal,

nId_Lider = @nId_Lider,

sDescripcion = @sDescripcion,

nId_Metodologia = @nId_Metodologia,

dFecha_Inicio = @dFecha_Inicio,

dFecha_Fin = @dFecha_Fin,

nUsuario_Update = @nUsuario_Update,

dDatetime_Update = GETDATE(),

sCodigo = ISNULL(@sCodigo, '-'),

sCodigo_SinPrefijo = @sCodigo_sinprefijo,

sNombre_SinPrefijo = @sNombre_SinPrefijo

WHERE nId_Requerimiento = @nId_Requerimiento;

  

  

SET @UpdatedId = 1;

SELECT @UpdatedId AS Result;

END TRY

BEGIN CATCH

SET @UpdatedId = 0;

SELECT @UpdatedId AS Result;

END CATCH

END;