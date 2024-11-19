```sql
DROP TABLE Titulo_Version
DROP TABLE Version
CREATE TABLE Version (
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
    ('3.10.0', 'FRONTEND_VERSION', 1, '2024-09-11', null, '2024-09-11', 172),
    ('3.12.1', 'FRONTEND_VERSION', 1, '2024-10-14', null, '2024-10-14', 172);
    ('3.13.0', 'FRONTEND_VERSION', 1, '2024-11-12', null, '2024-11-12', 172);


CREATE TABLE Titulo_Version (
    nId_ts INT IDENTITY(1,1) PRIMARY KEY,
    sTitle NVARCHAR(255) NOT NULL,
    sDescription NVARCHAR(MAX) NOT NULL,
    dFecha_Inicio DATE NOT NULL,
    dDatetime_Create DATETIME NOT NULL DEFAULT GETDATE(),
    dDatetime_Update DATETIME,
    nUsuario_Creador INT NOT NULL,
    nUsuario_Update INT,
    nId_v INT NOT NULL,
    nId_File INT NOT NULL,
    CONSTRAINT FK_ts_version FOREIGN KEY (nId_v) REFERENCES version(nId_v),
    CONSTRAINT FK_ts_files FOREIGN KEY (nId_File) REFERENCES Files(nId_File)
);


INSERT INTO Titulo_Version (sTitle, sDescription, dFecha_Inicio, dDatetime_Create, nUsuario_Creador, nId_v, nId_File)
VALUES 
    ('Conoce lo nuevo que hemos preparado para ti', 'Podrás estar al tanto de todas las actualizaciones de Smart con nuestro nuevo carrusel de Noticias. Solo haz clic en el botón "Noticias" y descubre todo lo que hemos preparado para ti.', '2024-09-16', '2024-09-16 19:54:03.467', 172, 1, 1543),
    ('Geolocalización en las marcaciones', 'Ahora, cada vez que marques tu asistencia, el equipo administrativo podrá ver tu ubicación geográfica en tiempo real.', '2024-09-17', '2024-09-17 19:22:27.700', 172, 1, 1544),
    ('Nuevo módulo de Comunicaciones', 'Crea boletines informativos de manera rápida usando nuestras plantillas pre-diseñadas. Programa su envío o mándalos al instante, ¡tú decides! Además, podrás guardarlos como borrador y continuar editándolos cuando lo necesites. *Disponible para el área administrativa.', '2024-10-07', '2024-10-07 21:32:00.687', 172, 1, 2365),
    ('Actualización del sistema', 'El sistema ha sido actualizado para mejorar su rendimiento. Los cambios ya están activos para una experiencia optimizada.', '2024-10-14', '2024-10-14 20:22:09.293', 172, 2, 2372);

------------------INSERCION DE NUEVA VERSION-----------------------------------


IF NOT EXISTS (
	SELECT * FROM VERSION WHERE sVersion = '3.13.0'
)
BEGIN
	BEGIN TRY
		BEGIN TRANSACTION
			INSERT INTO Version (sVersion, sNombre_Version, nEstado, dFecha_Inicio, dFecha_Fin, dDatetime_Creador, nUsuario_Creador)
			VALUES 
		    ('3.13.0', 'FRONTEND_VERSION', 1, '2024-11-12', null, '2024-11-12', 172);
			DECLARE @nId_Version INT;
		 	SET @nId_Version = SCOPE_IDENTITY();
			IF @nId_Version IS NOT NULL
				BEGIN	
			DECLARE @ID_FILE1 INT, @ID_FILE2 INT, @ID_FILE3 INT
			
			INSERT INTO  Files 
		    (nId_Entidad, sUrl_File, sEntidad, sTipo_File, nSize_File, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete) 
			VALUES 
		    (@nId_Version,'Versions/Imagenes/3-cierrepermiso-2024-14-11.jpg','Version','image/jpeg',2500,1,172,GETDATE(),null,null,null,null)
		    set @ID_FILE1 = SCOPE_IDENTITY();
			    
			    
			INSERT INTO FILES
		    (nId_Entidad, sUrl_File, sEntidad, sTipo_File, nSize_File, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
			VALUES
            (@nId_Version,'Versions/Imagenes/3-gestiontardanza-2024-14-11.jpg','Version','image/jpeg',2500,1,172,GETDATE(),null,null,null,null)
            set @ID_FILE2 = SCOPE_IDENTITY();
            
			INSERT INTO FILES
		    (nId_Entidad, sUrl_File, sEntidad, sTipo_File, nSize_File, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
			VALUES
            (@nId_Version,'Versions/Imagenes/3-masivocolaborador-2024-14-11.jpg','Version','image/jpeg',2500,1,172,GETDATE(),null,null,null,null)
            set @ID_FILE3 = SCOPE_IDENTITY();
			    

			INSERT INTO 
			Titulo_Version 
			    (sTitle, sDescription, dFecha_Inicio, dDatetime_Create, dDatetime_Update, nUsuario_Creador, nUsuario_Update, nId_v, nId_File)
				VALUES 
			    ('Gestiona las tardanzas', 'Gestiona tardanzas y puntualidad con una vista organizada y un indicador dinámico para un seguimiento completo de la asistencia de cada colaborador.', GETDATE(), GETDATE(), null, 172, null, @nId_Version, @ID_FILE1),
			    ('Cierre de Permisos', 'Gestiona permisos y recuperación de horas de manera eficiente.', GETDATE(), GETDATE(), null, 172, null, @nId_Version, @ID_FILE2),
			    ('Creación Masiva de Colaboradores', 'Simplifica la incorporación de nuevos colaboradores, agiliza la incorporación masiva de nuevos colaboradores con la función de "Inserción Masiva".', GETDATE(), GETDATE(), null, 172, null, @nId_Version, @ID_FILE3)
			END
		COMMIT TRANSACTION
	END TRY
	
	BEGIN CATCH
		ROLLBACK TRANSACTION
	END CATCH
END

```
