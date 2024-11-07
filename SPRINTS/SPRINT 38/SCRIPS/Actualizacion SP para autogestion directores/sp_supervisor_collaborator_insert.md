```sql
CREATE   PROCEDURE [dbo].[sp_supervisor_collaborator_insert]      
    @nId_Colaborador INT,      
    @nId_Supervisor INT,      
    @bPrincipal INT,      
    @nEstado INT,      
    @nUsuario_Creador INT,      
    @dDatetime_Creador DATETIME,      
    @nUsuario_Update INT = NULL,      
    @dDatetime_Update DATETIME = NULL,      
    @nUsuario_Delete INT = NULL,      
    @dDatetime_Delete DATETIME = NULL      
AS      
BEGIN      
    INSERT INTO colaboradores_supervisor (      
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
    )      
    VALUES (      
     @nId_Colaborador,      
       CASE               WHEN @nId_Supervisor = 0 THEN NULL               ELSE @nId_Supervisor           END,       
        @bPrincipal,      
        @nEstado,      
        @nUsuario_Creador,      
        @dDatetime_Creador,      
        @nUsuario_Update,      
        @dDatetime_Update,      
        @nUsuario_Delete,      
        @dDatetime_Delete      
    );      
END;
```