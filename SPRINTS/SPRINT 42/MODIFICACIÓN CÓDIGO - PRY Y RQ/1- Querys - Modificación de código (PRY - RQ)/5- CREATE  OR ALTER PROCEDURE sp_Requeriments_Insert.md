```SQL
CREATE OR ALTER PROCEDURE sp_Requeriments_Insert

@sNombre NVARCHAR(MAX),

@sDescripcion NVARCHAR(MAX),

@nEstado INT,

@nId_Proyecto INT,

@nUsuario_Creador INT,

@dDatetime_Creador DATETIME,

@nUsuario_Update INT,

@dDatetime_Update DATETIME,

@nUsuario_Delete INT,

@dDatetime_Delete DATETIME,

@dFecha_Inicio DATE,

@nId_Metodologia INT,

@dFecha_Fin DATE,

@dFecha_Modificacion_Estado DATETIME2,

@nId_Lider INT = 0,

@sCodigo NVARCHAR(MAX),

@sNombre_SinPrefijo nvarchar(max)

AS

BEGIN

SET NOCOUNT ON;

  

DECLARE @InsertedId INT;

--DECLARE @GeneratedCodigo NVARCHAR(MAX);

DECLARE @CodigoSinPrefijo NVARCHAR(MAX);

DECLARE @NombreFinal NVARCHAR(MAX);

  

IF @sCodigo IS NULL

BEGIN

SET @sCodigo = '-';

END

  

IF @sCodigo <> '-'

BEGIN

SET @NombreFinal = CONCAT(@sCodigo, ' - ', + @sNombre)

END

ELSE

BEGIN

SET @NombreFinal = CONCAT('RQ', ' - ', + @sNombre)

END

IF @sCodigo IS NOT NULL

BEGIN

SET @CodigoSinPrefijo = CASE

WHEN @sCodigo LIKE 'RQ %' THEN SUBSTRING(@sCodigo, 4, LEN(@sCodigo))

WHEN @sCodigo LIKE 'RQ%' THEN SUBSTRING(@sCodigo, 3, LEN(@sCodigo))

ELSE @sCodigo

END;

END

  

INSERT INTO Requerimientos

(

sNombre, sDescripcion, nEstado, nId_Proyecto, nUsuario_Creador, dDatetime_Creador,

nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, dFecha_Inicio, nId_Metodologia,

dFecha_Fin, dFecha_Modificacion_Estado, nId_Lider, sCodigo, sNombre_SinPrefijo,sCodigo_SinPrefijo

  

)

VALUES

(

@NombreFinal, @sDescripcion, @nEstado, @nId_Proyecto, @nUsuario_Creador, @dDatetime_Creador,

@nUsuario_Update, @dDatetime_Update, @nUsuario_Delete, @dDatetime_Delete, @dFecha_Inicio, @nId_Metodologia,

@dFecha_Fin, @dFecha_Modificacion_Estado, @nId_Lider, @sCodigo, @sNombre_SinPrefijo,

CASE

WHEN @sCodigo IS NULL THEN NULL

ELSE @CodigoSinPrefijo

END

);

  

SET @InsertedId = SCOPE_IDENTITY();

/*IF @sCodigo = '-'

BEGIN

UPDATE Requerimientos

SET sCodigo = '-',

sCodigo_SinPrefijo = '-'

WHERE nId_Requerimiento = @InsertedId;

END*/

  

  

/*IF @sCodigo IS NULL

BEGIN

--SET @GeneratedCodigo = CONCAT('RQ', CAST(@InsertedId AS NVARCHAR));

--SET @CodigoSinPrefijo = CAST(@InsertedId AS NVARCHAR);

  

UPDATE Requerimientos

SET sCodigo = '-',

sCodigo_SinPrefijo = '-'

WHERE nId_Requerimiento = @InsertedId;

END

ELSE

BEGIN

-- Usar el @sCodigo proporcionado

UPDATE Requerimientos

SET sCodigo = @sCodigo

WHERE nId_Requerimiento = @InsertedId;

END*/

  

SELECT @InsertedId AS 'nId_Requerimiento';

END;