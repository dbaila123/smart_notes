```sql
CREATE    PROCEDURE sp_massive_insert_collaborator       
  @PrimerNombre NVARCHAR(MAX),      
  @SegundoNombre NVARCHAR(MAX),      
  @ApellidoPaterno NVARCHAR(MAX),      
  @ApellidoMaterno NVARCHAR(MAX),      
  @FechaNacimiento DATETIME,      
  @Nacionalidad NVARCHAR(MAX),      
  @TipoDocumento NVARCHAR(MAX),      
  @NumeroDocumento NVARCHAR(MAX),      
  @Rol NVARCHAR(MAX),      
  @EmailEmpresarial NVARCHAR(MAX),      
  @Horario NVARCHAR(MAX),      
  @Sede NVARCHAR(MAX),      
  @Cargo NVARCHAR(MAX),      
  @Area NVARCHAR(MAX),      
  @FechaIngreso DATETIME,      
  @NombreEmergencia NVARCHAR(MAX),      
  @ParentescoEmergencia NVARCHAR(MAX),    
  @TelefonoEmergencia NVARCHAR(MAX)    
AS      
           
   IF EXISTS (      
 SELECT 1       
 FROM Documentos      
    WHERE sNumero_Documento = @NumeroDocumento     
   )    
       
        DECLARE @nId_cargo INT;      
   SELECT TOP 1 @nId_cargo = nId_cargo       
        FROM Cargos      
   WHERE sNombre LIKE '%' + @Cargo + '%';      
      
  DECLARE @nId_Nacionalidad INT;      
   SELECT TOP 1 @nId_Nacionalidad = nId_Pais       
        FROM Paises      
   WHERE sGentilicio LIKE '%' + @Nacionalidad + '%';      
BEGIN      
 -------------------------INSERCION DE PERSONAS--------------------------------      
     INSERT INTO Personas (      
  nId_Especialidad,      
  nId_Giro_Negocio,      
  nId_Ocupacion,      
  nId_Nacionalidad,      
  nId_Estado_Civil,      
  nId_Sexo,      
  nId_Tipo_Persona,      
  sPersona_Nombre,      
  sNombres,      
  sApe_Paterno,      
  sApe_Materno,      
  dFecha_Nacimiento,      
  nEstado,      
  nUsuario_Creador,      
  dDatetime_Creador,      
  nUsuario_Update,      
  dDatetime_Update,      
  nUsuario_Delete,      
  dDatetime_Delete,      
  sPrimer_Nombre,      
  sSegundo_Nombre      
  ) VALUES (      
  @nId_cargo,      
  null,      
  3,      
  @nId_Nacionalidad,      
  null,      
  null,      
  1,      
  CONCAT(@PrimerNombre, ' ', @SegundoNombre, ' ', @ApellidoPaterno, ' ', @ApellidoMaterno),      
  CONCAT(@PrimerNombre, '', @SegundoNombre),      
  @ApellidoPaterno,      
  @ApellidoMaterno,      
  @FechaNacimiento,      
  1,      
  172,      
  GETDATE(),      
  null,      
  null,      
  null,      
  null,      
  @PrimerNombre,      
  @SegundoNombre      
  );      
      
 DECLARE @nId_Persona INT;      
    SET @nId_Persona = SCOPE_IDENTITY();      
      
 IF @nId_Persona IS NOT NULL      
    BEGIN      
 -- Declara las variables que contendrán los valores de nombre y correos      
 DECLARE @EmailAlternativo NVARCHAR(255);      
      
 -- Verifica si ya existe el correo empresarial en la tabla Usuarios      
 IF EXISTS (SELECT 1 FROM Usuarios WHERE sEmail = @EmailEmpresarial)      
  BEGIN      
      -- Si el correo empresarial ya existe, genera el alternativo      
      SET @EmailAlternativo = LOWER(REPLACE(CONCAT(@PrimerNombre, '.', @ApellidoPaterno, LEFT(@ApellidoMaterno, 1), '@materiagris.pe'),'ñ','n'));      
  END      
 ELSE      
 BEGIN      
     -- Si el correo no existe, usa el correo original      
  SET @EmailEmpresarial = REPLACE(@EmailEmpresarial, 'ñ', 'n');      
     SET @EmailAlternativo = @EmailEmpresarial;      
 END      
        ---------------------------INSERCION DE USUARIOS--------------------------------      
        INSERT INTO Usuarios (      
            nId_Persona,      
            nTipo_Autenticacion,      
            sEmail,      
            sClave,      
            sUrl_Foto,      
            stoken_Recuperacion,      
            nEstado,      
            nUsuario_Creador,      
            dDateTime_Creador,      
            nUsuario_Update,      
            nUsuario_Delete,      
            dDatetime_Delete,      
            nTipo_Usuario      
        )       
        VALUES (      
            @nId_Persona,      
            1,      
            @EmailAlternativo,      
            '$2a$11$l/NKEc7t8KxGUGZoKIzH/O44qxOqk0ptVEhLeP5Guyoomu2F.FZs2',      
            NULL,      
            NULL,      
            1,      
            172,      
            GETDATE(),      
            NULL,      
            NULL,      
            NULL,      
            1      
        );      
      
  DECLARE @nId_Usuario INT;      
  SET @nId_Usuario = SCOPE_IDENTITY();      
        
  DECLARE @nId_Empresa INT;      
        SELECT @nId_Empresa = nId_Empresa       
        FROM v_persona_empresa       
        WHERE sNombre_Empresa = 'TECH SOURCE';      
      
  DECLARE @nId_Area INT;      
  SELECT TOP 1 @nId_Area = nId_Area       
        FROM Areas      
        WHERE sDescripcion LIKE '%' + @Area + '%';      
      
  IF @nId_Empresa IS NOT NULL      
  BEGIN      
  --------------------------------INSERCION A COLABORADORES---------------------------      
  INSERT INTO Colaboradores (      
    nId_Persona,      
    nId_Area,      
    nId_Empresa,      
    nTipo_Colaborador,      
    nTipo_Contratacion,      
    nTipo_Modalidad,      
    sUrl_Foto,      
    sMotivo_Cese,      
    nEps,      
    dFec_Ingreso,      
    nAutoriza_Tolerancia,      
    nEstado_Colaborador,      
    nEstado,      
    nUsuario_Creador,      
    dDatetime_Creador,      
    nUsuario_Update,      
    dDatetime_Update,      
    nUsuario_Delete,      
    dDatetime_Delete,      
    dFecha_Inicio,      
    dFecha_Fin,      
    bPersonal_Confianza      
    ) VALUES (      
    @nId_Persona,      
    @nId_Area,      
    @nId_Empresa,      
    1,      
    1,      
    1,      
    null,      
    null,      
    1,      
    @FechaIngreso,      
    null,      
    1,      
    1,      
    172,      
    GETDATE(),      
    null,      
    null,      
    null,      
    null,      
    null,      
    null,      
    0      
  );      
      
 DECLARE @nId_Colaborador INT;      
    SET @nId_Colaborador = SCOPE_IDENTITY();      
      
 DECLARE @nId_nTipoDocumento INT;      
 SELECT TOP 1 @nId_nTipoDocumento = sCodigo      
 FROM Configs      
 where sTabla = 'TIPO_DOCUMENTO' AND sDescripcion LIKE '%' + @TipoDocumento + '%';      
      
 DECLARE @nId_Horario INT;      
 SELECT TOP 1 @nId_Horario = nId_Horario       
    FROM Horarios      
    WHERE sDescripcion LIKE '%' + @Horario + '%';    
  
DECLARE @nId_Rol INT;  
SELECT TOP 1 @nId_Rol = nId_Rol  
FROM Roles  
Where sDescripcion LIKE '%' + @Rol + '%';    
      
 IF @nId_Colaborador IS NOT NULL      
    BEGIN      
    --------------------------------INSERCION COLABORADORES SUPERVISOR--------------------      
    INSERT INTO Colaboradores_supervisor (      
      nId_Colaborador,      
      nId_Supervisor,      
      bPrincipal,      
      nEstado,      
      nUsuario_Creador,      
      dDatetime_Creador,      
      nUsuario_Update,      
      dDatetime_Update,      
      nUsuario_Delete,      
      dDatetime_Delete      
      ) VALUES (      
      @nId_Colaborador,      
      1, --supervisor carlos durand      
      1,      
      1,      
      172,      
      GETDATE(),      
      null,      
      null,      
      null,      
      null      
    );      
    -----------------ROLES USUARIOS-----------      
    INSERT INTO roles_usuarios (      
      nId_Usuario,      
   nId_Rol,      
   nEstado,      
   nUsuario_Creador,      
   dDateTime_Creador,      
   nUsuario_Update,      
   dDatetime_Update,      
   nUsuario_Delete,      
   dDatetime_Delete      
    ) VALUES (      
      @nId_Usuario,      
   @nId_Rol,      
   1,      
   172,      
   GETDATE(),      
   null,      
   null,      
   null,      
   null      
    );      
   ---------------------------------INSERCION DE MAILS---------------------------------      
    INSERT INTO Emails(      
      nId_Persona,      
      nTipo_Email,      
      nTipo_Propietario,      
      sEmail,      
      nPrincipal,      
      nEstado,      
      nUsuario_creador,      
      dDatetime_Creador,      
      nUsuario_Update,      
      dDatetime_Update,      
      nUsuario_Delete,      
      dDatetime_Delete      
    ) VALUES (      
      @nId_Persona,      
   2,      
   0,      
   @EmailAlternativo,      
   1,      
   1,      
   172,      
   GETDATE(),      
   null,      
   null,      
   null,      
   null      
    );      
      
    INSERT INTO Colaboradores_Proyectos (      
      nId_Colaborador,      
      nId_Proyecto,      
      nPrincipal,      
      nEstado,      
      nUsuario_Creador,      
      dDatetime_Creador,      
      nUsuario_Update,      
      dDatetime_Update,      
      nUsuario_Delete,      
      dDatetime_Delete      
    ) VALUES (      
      @nId_Colaborador,      
      103,      
      1,      
      1,      
      172,      
      GETDATE(),      
      null,      
      null,      
      null,      
      null      
    );      
      
    INSERT INTO Cargos_Colaboradores (      
      nId_Cargo,      
   nId_Colaborador,      
   dFecha_Inicio,      
   dFecha_Fin,      
   nEstado,      
   nUsuario_Creador,      
   dDatetime_Creador,      
   nUsuario_Update,      
   dDatetime_Update,      
   nUsuario_Delete,      
   dDatetime_Delete      
    ) VALUES (      
      @nId_cargo,      
   @nId_Colaborador,      
   GETDATE(),      
   DATEADD(MONTH, 1, GETDATE()),      
   1,      
   172,      
   GETDATE(),      
   null,      
   null,      
   null,      
   null      
    );      
          
      
    INSERT INTO Horarios_Colaboradores (      
       nId_Horario,      
       nId_Colaborador,      
       nDiferencia_Horaria,      
       nEstado,      
       nUsuario_Creador,      
       dDatetime_Creador,      
       nUsuario_Update,      
       dDatetime_Update,      
       nUsuario_Delete,      
       dDatetime_Delete      
       ) VALUES (      
    @nId_Horario,      
       @nId_Colaborador,      
       0,      
       1,      
       172,      
       GETDATE(),      
       null,      
       null,      
       null,      
       null      
     );      
      
  INSERT INTO Sedes_Colaboradores (      
  nId_Colaborador,      
  nId_Sede,      
  nEstado,      
  nUsuario_Creador,      
  dDatetime_Creador,      
  nUsuario_Update,      
  dDatetime_Update,      
  nUsuario_Delete,      
  dDatetime_Delete      
  ) VALUES (      
  @nId_Colaborador,      
  1,      
  1,      
  172,      
  GETDATE(),      
  null,      
  null,      
  null,      
  null      
  );      
      
  INSERT INTO Team_colaboradors (      
    nId_Colaborador,      
    nId_Team,      
    nPrincipal,      
    nEstado,      
    nUsuario_Creador,      
    dDatetime_Creador,      
    nUsuario_Update,      
    dDatetime_Update,      
    nUsuario_Delete,      
    dDatetime_Delete      
  ) VALUES (      
    @nId_Colaborador,      
    1,      
    1,      
    1,      
    172,      
    GETDATE(),      
    null,      
    null,      
    null,      
    null      
  );      
  -----------------HABILITAR EL DASHBOARD DEL SMART---------------      
  INSERT INTO PreferenciasColaborador(      
    nTheme,      
    sCards_Orden,      
    sUrl,      
    nId_Colaborador,      
    dDatetime_Creador,      
    dDatetime_Update,      
    nUsuario_Creador,      
    nUsuario_Update,      
    nBackground_Theme,      
    nMode_Type_Theme      
  ) VALUES (      
    1,      
    '[[1, 3, 4], [2, 5], [6, 7], [8, 10, 11]]',      
    null,      
    @nId_Colaborador,      
    GETDATE(),      
    null,      
    172,      
    null,      
    1,      
    1      
  );      
      
  INSERT INTO Documentos (      
    nId_Persona,      
    nTipo_Documento,      
    sNumero_Documento,      
    sUrl_Documento,      
    nPrincipal,      
    dFecha_Emision,      
    dFecha_Caducidad,      
    nEstado,      
    nUsuario_Creador,      
    dDatetime_Creador,      
    nUsuario_Update,      
    dDatetime_Update,      
    nUsuario_Delete,      
    dDatetime_Delete      
  ) VALUES (      
    @nId_Persona,      
    @nId_nTipoDocumento,      
    @NumeroDocumento,      
    null,      
    1,      
    GETDATE(),      
    GETDATE(),      
    1,      
    172,      
    GETDATE(),      
    null,      
    null,      
    null,      
    null      
  );      
  END;      
      
  IF @NombreEmergencia IS NOT NULL       
  BEGIN      
  ----------------CONTACTO EMERGENCIA---------------      
  INSERT INTO Personas (      
    nId_Especialidad,      
    nId_Giro_Negocio,      
    nId_Ocupacion,      
    nId_Nacionalidad,      
    nId_Estado_Civil,      
    nId_Sexo,      
    nId_Tipo_Persona,      
    sPersona_Nombre,      
    sNombres,      
    sApe_Paterno,      
    sApe_Materno,      
    dFecha_Nacimiento,      
    nEstado,      
    nUsuario_Creador,      
    dDatetime_Creador,      
    nUsuario_Update,      
    dDatetime_Update,      
    nUsuario_Delete,      
    dDatetime_Delete,      
    sPrimer_Nombre,      
    sSegundo_Nombre      
    ) VALUES (      
    null,      
    null,      
    3,      
    1,      
    null,      
    null,      
    0,      
    @NombreEmergencia,      
    null,      
    null,      
    null,    
 null,    
    1,      
    172,      
    GETDATE(),      
    null,      
    null,      
    null,      
    null,      
    @NombreEmergencia,      
    null      
    );      
      
 DECLARE @nId_Persona_Emergencia INT;      
    SET @nId_Persona_Emergencia = SCOPE_IDENTITY();      
      
 DECLARE @nId_Parentesco INT;      
 SELECT TOP 1 @nId_Parentesco = sCodigo       
    FROM Configs      
    WHERE sTabla = 'parentesco' and sDescripcion LIKE '%' + @ParentescoEmergencia + '%';      
      
 IF @nId_Persona_Emergencia IS NOT NULL      
    BEGIN      
 ----------------CONTACTO EMERGENCIA---------------      
   INSERT INTO Contactos (      
      nId_Persona,      
      nId_Persona_Contacto,      
      nPrincipal,      
      sComentario,      
      nOrden,      
      sCargo,      
      nParentesco,      
      nEstado,      
      nUsuario_Creador,      
      dDatetime_Creador,      
      nUsuario_Update,      
      dDatetime_Update,      
      nUsuario_Delete,      
      dDatetime_Delete      
      ) values (      
      @nId_Persona,      
      @nId_Persona_Emergencia,      
      1,      
      null,      
      1,      
      null,      
      @nId_Parentesco,      
      1,      
      172,      
      GETDATE(),      
      null,      
      null,      
      null,      
    null      
      );     
       
   Insert into Telefonos(    
   nId_Persona,     
   nId_Pais,     
   nTipo_Propietario,     
   nTipo_Telefono,     
   sPrefijo_Area_Local,     
   sTelefono,nPrincipal,    
   sAnexo1,    
   sAnexo2,    
   nEstado,    
   nUsuario_Creador,    
   dDatetime_Creador,    
   nUsuario_Update,    
   dDatetime_Update,    
   nUsuario_Delete,    
   dDatetime_Delete    
   ) VALUES (    
   @nId_Persona_Emergencia,    
   1,    
   0,    
   2,    
   null,    
   @TelefonoEmergencia,    
   1,    
   null,    
   null,    
   1,    
   172,    
   GETDATE(),    
   null,    
   null,    
   null,    
   null    
   );    
    
    END;      
  END;      
   END;      
  END;      
END; 
```