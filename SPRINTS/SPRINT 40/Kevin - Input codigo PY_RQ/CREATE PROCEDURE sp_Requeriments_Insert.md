```SQL
CREATE PROCEDURE sp_Requeriments_Insert

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

@sCodigo NVARCHAR(MAX)

AS

BEGIN

SET NOCOUNT ON;

  

DECLARE @InsertedId INT;

DECLARE @GeneratedCodigo NVARCHAR(MAX);

  

INSERT INTO Requerimientos

(

sNombre, sDescripcion, nEstado, nId_Proyecto, nUsuario_Creador, dDatetime_Creador,

nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete, dFecha_Inicio, nId_Metodologia,

dFecha_Fin, dFecha_Modificacion_Estado, nId_Lider, sCodigo

)

VALUES

(

@sNombre, @sDescripcion, @nEstado, @nId_Proyecto, @nUsuario_Creador, @dDatetime_Creador,

@nUsuario_Update, @dDatetime_Update, @nUsuario_Delete, @dDatetime_Delete, @dFecha_Inicio, @nId_Metodologia,

@dFecha_Fin, @dFecha_Modificacion_Estado, @nId_Lider, @sCodigo

);

  

SET @InsertedId = SCOPE_IDENTITY();

  

IF @sCodigo IS NULL

BEGIN

SET @GeneratedCodigo = CONCAT('RQ-', CAST(@InsertedId AS NVARCHAR));

  

UPDATE Requerimientos

SET sCodigo = @GeneratedCodigo

WHERE nId_Requerimiento = @InsertedId;

END

ELSE

BEGIN

SET @GeneratedCodigo = @sCodigo;

END

  

SELECT @InsertedId AS 'nId_Requerimiento';

END;
```