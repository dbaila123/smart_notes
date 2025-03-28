```SQL
CREATE TABLE tenant0001.dbo.collaborators_certificate (

    nId_Certificado int IDENTITY(1,1) NOT NULL,

    nId_Colaborador int NOT NULL,

    sNombre nvarchar(255) COLLATE Modern_Spanish_CI_AS NULL,

    sUrl nvarchar(MAX) COLLATE Modern_Spanish_CI_AS NULL,

    dFecha_Inicio date NOT NULL,

    dFecha_Expiracion date NOT NULL,

    nCantidad_Horas int NOT NULL,

    sDescripcion nvarchar(500) COLLATE Modern_Spanish_CI_AS NULL,

    nEstado int NOT NULL,

    nUsuario_Creador int NOT NULL,

    dDatetime_Creador datetime NOT NULL,

    nUsuario_Update int NULL,

    dDatetime_Update datetime NULL,

    nUsuario_Delete int NULL,

    dDatetime_Delete datetime NULL,

    CONSTRAINT PK_Certificado_Colaboradores PRIMARY KEY (nId_Certificado)

)