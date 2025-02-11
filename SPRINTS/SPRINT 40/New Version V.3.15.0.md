```sql
IF NOT EXISTS (
	SELECT * FROM VERSION WHERE sVersion = '3.15.0'
)
BEGIN
	BEGIN TRY
		BEGIN TRANSACTION
			INSERT INTO Version (sVersion, sNombre_Version, nEstado, dFecha_Inicio, dFecha_Fin, dDatetime_Creador, nUsuario_Creador)
			VALUES 
		    ('3.15.0', 'FRONTEND_VERSION', 1, '2024-01-17', null, '2025-01-17', 172);
			DECLARE @nId_Version INT;
		 	SET @nId_Version = SCOPE_IDENTITY();
			IF @nId_Version IS NOT NULL
				BEGIN	
			DECLARE @ID_FILE1 INT, @ID_FILE2 INT
			
			INSERT INTO  Files 
		    (nId_Entidad, sUrl_File, sEntidad, sTipo_File, nSize_File, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete) 
			VALUES 
		    (@nId_Version,'Version/Imagenes/5-codigopryrq-2025-01-17.jpg','Version','image/jpeg',2500,1,172,GETDATE(),null,null,null,null)
		    set @ID_FILE1 = SCOPE_IDENTITY();
			    
			INSERT INTO FILES
		    (nId_Entidad, sUrl_File, sEntidad, sTipo_File, nSize_File, nEstado, nUsuario_Creador, dDatetime_Creador, nUsuario_Update, dDatetime_Update, nUsuario_Delete, dDatetime_Delete)
			VALUES
            (@nId_Version,'Version/Imagenes/5-reporteindicadores-2025-01-17.jpg','Version','image/jpeg',2500,1,172,GETDATE(),null,null,null,null)
            set @ID_FILE2 = SCOPE_IDENTITY();    

			INSERT INTO 
			Titulo_Version 
			    (sTitle, sDescription, dFecha_Inicio, dDatetime_Create, dDatetime_Update, nUsuario_Creador, nUsuario_Update, nId_v, nId_File)
				VALUES 
			    ('Código Único para PRY y RQ', 'Ahora puedes asignar un código único a cada proyecto o requerimiento. Si decides no ingresarlo, SMART lo generará automáticamente, asegurando siempre la unicidad del código.', GETDATE(), GETDATE(), null, 172, null, @nId_Version, @ID_FILE1),
			    ('Nuevo Botón de Reporte', 'Se ha añadido un botón de reporte donde pondrás visualizar las horas consumidas clasificadas por tipo y la comparación entre el tiempo estimado y el tiempo realmente usado.', GETDATE(), GETDATE(), null, 172, null, @nId_Version, @ID_FILE2)
			END
		COMMIT TRANSACTION
	END TRY
	
	BEGIN CATCH
		ROLLBACK TRANSACTION
	END CATCH
END
```
