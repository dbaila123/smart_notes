```sql
CREATE PROCEDURE sp_smart_areas_update

(

@nId_Area INT,

@sDescripcion NVARCHAR(MAX) = NULL,

@sComentario NVARCHAR(MAX) = NULL,

@nEstado INT = NULL,

@nUsuario_Update INT,

@dDatetime_Update DATETIME

)

AS

BEGIN

IF @nId_Area IS NULL

BEGIN

RAISERROR('El ID del área no puede ser NULL.', 16, 1);

RETURN;

END

  

UPDATE Areas

SET

sDescripcion = COALESCE(@sDescripcion, sDescripcion), -- Si es NULL, mantiene el valor actual

sComentario = COALESCE(@sComentario, sComentario),

nEstado = COALESCE(@nEstado, nEstado),

nUsuario_Update = @nUsuario_Update,

dDatetime_Update = @dDatetime_Update

WHERE

nId_Area = @nId_Area;

  

IF @@ROWCOUNT = 0

BEGIN

RAISERROR('No se encontró el área con el ID especificado.', 16, 1);

END

ELSE

BEGIN

SELECT 'Área actualizada correctamente' AS Mensaje, @nId_Area AS nId_Area;

END

END;
```