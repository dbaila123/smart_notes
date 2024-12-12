
Nota: colocar antes del "colaboradores_suggest"
```sql
  ------ ACTUALIZAR INFORMACION PARA COLABORADRES MASIVOS        
  IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'collaborator_self_update_information') = 1       
   BEGIN       
    INSERT INTO @configuraciones      
     select (col.nId_Colaborador) as sCodigo,      
     null          as sDescripcion,      
     'COLLABORATOR-SELF-UPDATE'  as sTipo_Configuracion      
    FROM Colaboradores col      
     inner join Usuarios us on us.nId_Persona = col.nId_Persona      
    WHERE col.bSelfUpdate = 1 AND us.nId_Usuario = @id_usuario    
   END      
  
```
