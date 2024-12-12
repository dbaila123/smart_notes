```sql
CREATE  OR ALTER PROCEDURE [dbo].[sp_collaborator_update]    
    @nId_Colaborador INT,    
    @nId_Persona INT = NULL,    
    @nId_Area INT = NULL,    
    @nId_Empresa INT = NULL,    
    @nTipo_Colaborador INT = NULL,    
    @nTipo_Contratacion INT = NULL,    
    @dFecha_Inicio DATETIME = NULL,    
    @dFecha_Fin DATETIME = NULL,    
    @nTipo_Modalidad INT = NULL,    
    @sUrl_Foto NVARCHAR(MAX) = NULL,    
    @sMotivo_Cese NVARCHAR(MAX) = NULL,    
    @nEps INT = NULL,    
    @dFec_Ingreso DATETIME = NULL,    
    @nAutoriza_Tolerancia INT = NULL,    
    @nEstado_Colaborador INT = NULL,    
    @nEstado INT = NULL,    
    @nUsuario_Creador INT = NULL,    
    @dDatetime_Creador DATETIME = NULL,    
    @nUsuario_Update INT = NULL,    
    @dDatetime_Update DATETIME = NULL,    
    @nUsuario_Delete INT = NULL,    
    @dDatetime_Delete DATETIME = NULL,    
    @Null_For_nUsuario_dDatetime_Delete_And_sMotivo_Cese BIT = 0    
AS    
BEGIN    
    
 --cargo    
 DECLARE @nId_Cargo INT= (SELECT nId_Cargo FROM Cargos_Colaboradores WHERE nId_Colaborador = @nId_Colaborador)    
 DECLARE @nEstado_Cargo INT = (SELECT nEstado_Cargos FROM cargos WHERE nId_Cargo = @nId_Cargo)    
    
 IF @nEstado_Cargo = 2    
 BEGIN    
  UPDATE cargos    
  SET nEstado_Cargos = 1    
  WHERE nId_Cargo = @nId_Cargo    
 END    
    
    UPDATE Colaboradores    
    SET    
    nId_Persona = ISNULL(@nId_Persona, nId_Persona),    
    nId_Area = ISNULL(@nId_Area, nId_Area),    
    nId_Empresa = ISNULL(@nId_Empresa, nId_Empresa),    
    nTipo_Colaborador = ISNULL(@nTipo_Colaborador, nTipo_Colaborador),    
    nTipo_Contratacion = ISNULL(@nTipo_Contratacion, nTipo_Contratacion),    
    dFecha_Inicio = ISNULL(@dFecha_Inicio, dFecha_Inicio),    
    dFecha_Fin = ISNULL(@dFecha_Fin, dFecha_Fin),    
    nTipo_Modalidad = ISNULL(@nTipo_Modalidad, nTipo_Modalidad),    
    sUrl_Foto = ISNULL(@sUrl_Foto, sUrl_Foto),    
    sMotivo_Cese = CASE    
        WHEN @Null_For_nUsuario_dDatetime_Delete_And_sMotivo_Cese = 0 THEN ISNULL(@sMotivo_Cese, sMotivo_Cese)    
        WHEN @Null_For_nUsuario_dDatetime_Delete_And_sMotivo_Cese = 1 THEN NULL    
          END,    
    nEps = ISNULL(@nEps, nEps),    
    dFec_Ingreso = ISNULL(@dFec_Ingreso, dFec_Ingreso),    
    nAutoriza_Tolerancia = ISNULL(@nAutoriza_Tolerancia, nAutoriza_Tolerancia),    
    nEstado_Colaborador = ISNULL(@nEstado_Colaborador, nEstado_Colaborador),    
    nEstado = ISNULL(@nEstado, nEstado),    
    nUsuario_Creador = ISNULL(@nUsuario_Creador, nUsuario_Creador),    
    dDatetime_Creador = ISNULL(@dDatetime_Creador, dDatetime_Creador),    
    nUsuario_Update = ISNULL(@nUsuario_Update, nUsuario_Update),    
    dDatetime_Update = ISNULL(@dDatetime_Update, dDatetime_Update),    
    nUsuario_Delete = CASE    
         WHEN @Null_For_nUsuario_dDatetime_Delete_And_sMotivo_Cese = 0 THEN ISNULL(@nUsuario_Delete, nUsuario_Delete)    
         WHEN @Null_For_nUsuario_dDatetime_Delete_And_sMotivo_Cese = 1 THEN NULL    
          END,    
 dDatetime_Delete = CASE    
         WHEN @Null_For_nUsuario_dDatetime_Delete_And_sMotivo_Cese = 0 THEN ISNULL(@dDatetime_Delete, dDatetime_Delete)    
         WHEN @Null_For_nUsuario_dDatetime_Delete_And_sMotivo_Cese = 1 THEN NULL    
           END,  
 bselfupdate = null  
    WHERE nId_Colaborador = @nId_Colaborador;    
END;
```
