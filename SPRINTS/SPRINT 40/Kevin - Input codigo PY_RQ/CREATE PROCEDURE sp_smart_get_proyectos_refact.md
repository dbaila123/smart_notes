```SQL
ALTER OR ALTER PROCEDURE [dbo].[sp_smart_insert_or_update_proyecto]

(

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

DECLARE

@code VARCHAR(MAX),

@message VARCHAR(MAX), --MENSAJE DE ERROR

@count INT,

@currentProject VARCHAR(MAX),

@IdProyecto INT,

@GeneratedCodigo VARCHAR(MAX);

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

WHERE p.sNombre = @sNombre

AND cp.nId_Coordinador = @nId_Coordinador

AND c.nEstado = 1

AND c.sTabla = 'TIPO_SERVICIO'

);

IF @count = 0

BEGIN

-- Generación del código del proyecto

IF @sCodigo IS NULL

BEGIN

SET @GeneratedCodigo = (SELECT ISNULL(MAX(nId_Proyecto), 0) + 1 FROM Proyectos);

--SET @GeneratedCodigo = CONCAT('PRY ', (SELECT ISNULL(MAX(nId_Proyecto), 0) + 1 FROM Proyectos));

END

ELSE

BEGIN

SET @GeneratedCodigo = @sCodigo;

END
-- Insertar proyecto

INSERT INTO Proyectos

(

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

VALUES

(

@sNombre_Sin_Prefijo,

@sNombre,

@GeneratedCodigo,

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
SET @IdProyecto = (SELECT SCOPE_IDENTITY());

-- Insertar en Coordinadores_Proyectos

INSERT INTO Coordinadores_Proyectos

(

nId_Coordinador,

nId_Proyecto,

nPrincipal,

nEstado,

nUsuario_Creador,

dDatetime_Creador

)

VALUES

(

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

SET @message = 'El ' + @sNombre + ' ya tiene un coordinador con el mismo nombre.';

SELECT @code AS 'Code', @message AS 'Message', @IdProyecto AS 'IdProyecto';

END

END

-- Acción de actualización

IF @action = 2

BEGIN



SET @code = '200';

SET @message = 'Proyecto actualizado correctamente.';

SET @currentProject = (

SELECT p.sNombre

FROM Proyectos p

JOIN Configs c ON c.sCodigo = p.sId_Tipo_Servicio

JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto

WHERE p.sNombre = @sNombre

and p.nid_proyecto = @nId_Proyecto

AND cp.nId_Coordinador = @nId_Coordinador

AND c.nEstado = 1

AND c.sTabla = 'TIPO_SERVICIO'

);


DECLARE @nId_Proyecto_out INT;


SET @nId_Proyecto_out = (

SELECT p.nId_Proyecto

FROM Proyectos p

JOIN Configs c ON c.sCodigo = p.sId_Tipo_Servicio

JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto

WHERE p.sNombre = @sNombre

and p.nid_proyecto = @nId_Proyecto

AND cp.nId_Coordinador = @nId_Coordinador

AND c.nEstado = 1

AND c.sTabla = 'TIPO_SERVICIO'

);


IF (@currentProject = @sNombre AND @nId_Proyecto != @nId_Proyecto_out)
BEGIN	
	SET @code = '400';
	
	SET @message = 'El ' + @sNombre + ' ya tiene un coordinador con el mismo nombre.';
	
	SELECT @code AS 'Code', @message AS 'Message', @nId_Proyecto AS 'IdProyecto';

END
ELSE
BEGIN
	DECLARE @CodigoP VARCHAR(MAX) = NULL
	set @CodigoP = iif(@sCodigo IS NULL, CONCAT(@sPrefijo,cast(@nId_Proyecto as varchar)), @sCodigo)
END
-- Actualización de proyecto


UPDATE Proyectos

SET

sNombre = @sNombre,

sNombre_Proyecto_Sin_Prefijo = @sNombre_Sin_Prefijo,

sPrefijo = @sPrefijo_Codigo,

sCodigo = @CodigoP,

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

SET

nId_Coordinador = @nId_Coordinador,

nUsuario_Update = @nId_Usuario,

dDatetime_Update = DATEADD(HOUR, -5, GETDATE())

WHERE nId_Proyecto = @nId_Proyecto;

SELECT @code AS 'Code', @message AS 'Message', @nId_Proyecto AS 'IdProyecto';

END

END TRY

BEGIN CATCH

SELECT ERROR_SEVERITY() AS 'Code', ERROR_MESSAGE() AS 'Message', @nId_Proyecto AS 'IdProyecto';

END CATCH