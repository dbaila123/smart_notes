```sql
CREATE TABLE version (
    nId_v INT IDENTITY(1,1) PRIMARY KEY,
    sVersion NVARCHAR(50) NOT NULL,
    sNombre_Version NVARCHAR(100) NOT NULL,
    nEstado BIT NOT NULL,
    dFecha_Inicio DATE NOT NULL,
    dFecha_Fin DATE,
    dDatetime_Creador DATETIME NOT NULL DEFAULT GETDATE(),
    dDatetime_Update DATETIME,
    nUsuario_Creador INT NOT NULL,
    nUsuario_Update INT
);

INSERT INTO Version (sVersion, sNombre_Version, nEstado, dFecha_Inicio, dFecha_Fin, dDatetime_Creador, nUsuario_Creador)
VALUES 
    ('3.10.0', 'FRONTEND_VERSION', 1, '2024-09-11', '2024-09-11', '2024-09-11 16:38:09.347', 172),
    ('3.12.1', 'FRONTEND_VERSION', 1, '2024-10-14', '2024-10-14', '2024-10-14 00:00:00.000', 172);



CREATE TABLE Titulo_Version (
    nId_ts INT IDENTITY(1,1) PRIMARY KEY,          -- Autoincremental primary key
    sTitle NVARCHAR(255) NOT NULL,                 -- Title
    sDescription NVARCHAR(MAX) NOT NULL,           -- Description
    dFecha_Inicio DATE NOT NULL,                   -- Start date
    dDatetime_Create DATETIME NOT NULL DEFAULT GETDATE(), -- Creation datetime
    dDatetime_Update DATETIME,                     -- Update datetime (nullable)
    nUsuario_Creador INT NOT NULL,                 -- ID of the creator user
    nUsuario_Update INT,                           -- ID of the last update user (nullable)
    nId_v INT NOT NULL,                            -- Foreign key to `version` table
    nId_File INT NOT NULL,                         -- Foreign key to `Files` table
    CONSTRAINT FK_ts_version FOREIGN KEY (nId_v) REFERENCES version(nId_v),
    CONSTRAINT FK_ts_files FOREIGN KEY (nId_File) REFERENCES Files(nId_File)
);


INSERT INTO Titulo_Version (sTitle, sDescription, dFecha_Inicio, dDatetime_Create, nUsuario_Creador, nId_v, nId_File)
VALUES 
    ('Conoce lo nuevo que hemos preparado para ti', 'Podrás estar al tanto de todas las actualizaciones de Smart con nuestro nuevo carrusel de Noticias. Solo haz clic en el botón "Noticias" y descubre todo lo que hemos preparado para ti.', '2024-09-16', '2024-09-16 19:54:03.467', 172, 1, 1543),
    ('Geolocalización en las marcaciones', 'Ahora, cada vez que marques tu asistencia, el equipo administrativo podrá ver tu ubicación geográfica en tiempo real.', '2024-09-17', '2024-09-17 19:22:27.700', 172, 1, 1544),
    ('Nuevo módulo de Comunicaciones', 'Crea boletines informativos de manera rápida usando nuestras plantillas pre-diseñadas. Programa su envío o mándalos al instante, ¡tú decides! Además, podrás guardarlos como borrador y continuar editándolos cuando lo necesites. *Disponible para el área administrativa.', '2024-10-07', '2024-10-07 21:32:00.687', 172, 1, 2365),
    ('Actualización del sistema', 'El sistema ha sido actualizado para mejorar su rendimiento. Los cambios ya están activos para una experiencia optimizada.', '2024-10-14', '2024-10-14 20:22:09.293', 172, 2, 2372);


```