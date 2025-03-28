```SQL
CREATE PROCEDURE [dbo].[sp_certificate_insert]

    @nId_Colaborador INT,

    @sNombre NVARCHAR(255),

    @sUrl NVARCHAR(MAX),

    @dFecha_Inicio DATE,

    @dFecha_Expiracion DATE,

    @sCantidad_Horas INT = NULL,

    @sDescripcion NVARCHAR(500),

    @nEstado INT,

    @nUsuario_Creador INT,

    @dDatetime_Creador DATETIME,

    @nUsuario_Update INT = NULL,

    @dDatetime_Update DATETIME = NULL,

    @nUsuario_Delete INT = NULL,

    @dDatetime_Delete DATETIME = NULL

AS

BEGIN

    SET NOCOUNT ON;

  

    INSERT INTO tenant0001.dbo.collaborators_certificate (

        nId_Colaborador,

        sNombre,

        sUrl,

        dFecha_Inicio,

        dFecha_Expiracion,

        sCantidad_Horas,

        sDescripcion,

        nEstado,

        nUsuario_Creador,

        dDatetime_Creador,

        nUsuario_Update,

        dDatetime_Update,

        nUsuario_Delete,

        dDatetime_Delete

    )

    VALUES (

        @nId_Colaborador,

        @sNombre,

        @sUrl,

        @dFecha_Inicio,

        @dFecha_Expiracion,

        @sCantidad_Horas,

        @sDescripcion,

        @nEstado,

        @nUsuario_Creador,

        @dDatetime_Creador,

        @nUsuario_Update,

        @dDatetime_Update,

        @nUsuario_Delete,

        @dDatetime_Delete

    );

END;