```SQL
CREATE OR ALTER PROCEDURE sp_smart_insert_or_update_proyecto (

@nId_Usuario INT,

@nId_Proyecto INT = 0,

@sNombre_Sin_Prefijo VARCHAR(MAX),

@sNombre VARCHAR(MAX),

@sCodigo VARCHAR(MAX),

@sPrefijo VARCHAR(50),

@sPrefijo_Codigo VARCHAR(10),

@sId_Tipo_Servicio VARCHAR(MAX),

@nId_Lider INT,

@nId_Director INT,

@nId_Coordinador INT,

@sDescripcion VARCHAR(MAX),

@dFecha_Inicio DATE,

@dFecha_Fin DATE,

@nTipo_Acceso INT = 0,

@action INT

)

AS

BEGIN TRY

DECLARE @code VARCHAR(MAX);

DECLARE @message VARCHAR(MAX); -- MENSAJE DE ERROR

DECLARE @count INT;

DECLARE @currentProject VARCHAR(MAX);

DECLARE @IdProyecto INT;

DECLARE @GeneratedCodigo VARCHAR(MAX);

DECLARE @NombreFinal VARCHAR(MAX);

IF @sCodigo <> '-'

BEGIN

SET @NombreFinal = CONCAT(@sCodigo, ' - ', + @sNombre)

END

ELSE

BEGIN

IF @sPrefijo_Codigo = '1'

SET @NombreFinal = CONCAT('PRY', ' - ', + @sNombre)

ELSE IF @sPrefijo_Codigo = '2'

SET @NombreFinal = CONCAT('MAN', ' - ', + @sNombre)

ELSE

SET @NombreFinal = @sNombre

END

  

-- Acción de inserción

IF @action = 1

BEGIN

SET @code = '200';

SET @message = 'Proyecto registrado correctamente.';

  

SET @count = (

SELECT COUNT(*)

FROM Proyectos p

JOIN Configs c ON c.sCodigo = p.sId_Tipo_Servicio

JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto

WHERE p.sNombre = @NombreFinal

AND cp.nId_Coordinador = @nId_Coordinador

AND c.nEstado = 1

AND c.sTabla = 'TIPO_SERVICIO'

);

  

IF @count = 0

BEGIN

/*-- Generación del código del proyecto

IF @sCodigo IS NULL

BEGIN

SET @GeneratedCodigo = (

SELECT CONCAT(c.sDescripcion, ISNULL(MAX(p.nId_Proyecto), 0) + 1) AS ProyectoIdDescripcion

FROM Proyectos p

LEFT JOIN Configs c ON p.sPrefijo = c.sCodigo

WHERE c.sTabla = 'PREFIJO_PROYECTO'

AND c.sCodigo = @sPrefijo_Codigo

GROUP BY c.sDescripcion

);

END

ELSE

SET @GeneratedCodigo = @sCodigo;*/

IF @sCodigo IS NULL

BEGIN

SET @sCodigo = '-';

END

  

-- Insertar proyecto

INSERT INTO Proyectos (

sNombre_Proyecto_Sin_Prefijo,

sNombre,

sCodigo,

sPrefijo,

sDescripcion,

sId_Tipo_Servicio,

nId_Lider,

nId_Director,

sEstado_Proyecto,

nEstado,

dFecha_Inicio,

dFecha_Fin,

nUsuario_Creador,

dDatetime_Creador,

nTipo_Acceso

)

VALUES (

@sNombre_Sin_Prefijo,

@NombreFinal,

@sCodigo,

@sPrefijo_Codigo,

@sDescripcion,

@sId_Tipo_Servicio,

@nId_Lider,

@nId_Director,

1, -- estado activo

1, -- estado activo

@dFecha_Inicio,

@dFecha_Fin,

@nId_Usuario,

DATEADD(HOUR, -5, GETDATE()),

@nTipo_Acceso

);

  

SET @IdProyecto = SCOPE_IDENTITY();

  

-- Insertar en Coordinadores_Proyectos

INSERT INTO Coordinadores_Proyectos (

nId_Coordinador,

nId_Proyecto,

nPrincipal,

nEstado,

nUsuario_Creador,

dDatetime_Creador

)

VALUES (

@nId_Coordinador,

@IdProyecto,

0, -- No principal

1, -- Estado activo

@nId_Usuario,

DATEADD(HOUR, -5, GETDATE())

);

  

SELECT @code AS 'Code', @message AS 'Message', @IdProyecto AS 'IdProyecto';

END

ELSE

BEGIN

SET @code = '400';

SET @message = 'El ' + @NombreFinal + ' ya tiene un coordinador con el mismo nombre.';

SELECT @code AS 'Code', @message AS 'Message', @IdProyecto AS 'IdProyecto';

END

END

  

-- Acción de actualización

IF @action = 2

BEGIN

SET @code = '200';

SET @message = 'Proyecto actualizado correctamente.';

  

SET @currentProject = (

SELECT COUNT(*)

FROM Proyectos p

JOIN Configs c ON c.sCodigo = p.sId_Tipo_Servicio

JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto

WHERE p.sNombre = @NombreFinal

AND cp.nId_Coordinador = @nId_Coordinador

AND c.nEstado = 1

AND c.sTabla = 'TIPO_SERVICIO'

AND p.nId_Proyecto <> @nId_Proyecto

);

/*DECLARE @nId_Proyecto_out INT;

  

SET @nId_Proyecto_out = (

SELECT p.nId_Proyecto

FROM Proyectos p

JOIN Configs c ON c.sCodigo = p.sId_Tipo_Servicio

JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto

WHERE p.sNombre = @NombreFinal

AND p.nid_proyecto = @nId_Proyecto

AND cp.nId_Coordinador = @nId_Coordinador

AND c.nEstado = 1

AND c.sTabla = 'TIPO_SERVICIO'

);*/

  

IF @currentProject <> 0

BEGIN

SET @code = '400';

SET @message = 'El ' + @NombreFinal + ' ya tiene un coordinador con el mismo nombre.';

SELECT @code AS 'Code', @message AS 'Message', @nId_Proyecto AS 'IdProyecto';

END

ELSE

BEGIN

/*DECLARE @CodigoP VARCHAR(MAX) = NULL;

DECLARE @NewPrefijo VARCHAR(MAX);

SET @NewPrefijo = (SELECT c.sDescripcion FROM Configs c WHERE sTabla = 'PREFIJO_PROYECTO' AND sCodigo = @sPrefijo_Codigo);

SET @CodigoP = IIF(@sCodigo IS NULL, CONCAT(@NewPrefijo, CAST(@nId_Proyecto AS VARCHAR)), @sCodigo);*/

IF @sCodigo IS NULL

BEGIN

SET @sCodigo = '-';

END

  

-- Actualización de proyecto

UPDATE Proyectos

SET sNombre = @NombreFinal,

sNombre_Proyecto_Sin_Prefijo = @sNombre_Sin_Prefijo,

sPrefijo = @sPrefijo_Codigo,

sCodigo = @sCodigo,

sDescripcion = @sDescripcion,

sId_Tipo_Servicio = @sId_Tipo_Servicio,

nId_Lider = @nId_Lider,

nId_Director = @nId_Director,

dFecha_Inicio = @dFecha_Inicio,

dFecha_Fin = @dFecha_Fin,

nUsuario_Update = @nId_Usuario,

nTipo_Acceso = @nTipo_Acceso,

dDatetime_Update = DATEADD(HOUR, -5, GETDATE())

WHERE nId_Proyecto = @nId_Proyecto;

  

-- Actualización de Coordinadores_Proyectos

UPDATE Coordinadores_Proyectos

SET nId_Coordinador = @nId_Coordinador,

nUsuario_Update = @nId_Usuario,

dDatetime_Update = DATEADD(HOUR, -5, GETDATE())

WHERE nId_Proyecto = @nId_Proyecto;

  

SELECT @code AS 'Code', @message AS 'Message', @nId_Proyecto AS 'IdProyecto';

END

END

END TRY

BEGIN CATCH

SELECT ERROR_SEVERITY() AS 'Code', ERROR_MESSAGE() AS 'Message', @nId_Proyecto AS 'IdProyecto';

END CATCH;