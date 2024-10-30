```sql
CREATE PROCEDURE sp_smart_areas_insert

@nId_Area INT,

@sDescripcion NVARCHAR(MAX),

@sComentario NVARCHAR(MAX),

@nEstado INT,

@nUsuario_Creador INT,

@dDatetime_Creador DATETIME,

@nUsuario_Update INT = NULL,

@dDatetime_Update DATETIME = NULL,

@nUsuario_Delete INT = NULL,

@dDatetime_Delete DATETIME = NULL

AS

BEGIN

INSERT INTO Areas (

sDescripcion,

sComentario,

nEstado,

nUsuario_Creador,

dDatetime_Creador,

nUsuario_Update,

dDatetime_Update,

nUsuario_Delete,

dDatetime_Delete

)

VALUES (

@sDescripcion,

@sComentario,

@nEstado,

@nUsuario_Creador,

@dDatetime_Creador,

@nUsuario_Update,

@dDatetime_Update,

@nUsuario_Delete,

@dDatetime_Delete

)

  

SELECT SCOPE_IDENTITY() AS nId_Area

  

END
```