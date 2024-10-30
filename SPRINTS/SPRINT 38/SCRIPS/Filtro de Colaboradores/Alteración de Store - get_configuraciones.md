Cambio en slug de sedes:
```sql
IF (select count(sConfiguration) from @configurations_request where sConfiguration = 'sedes_suggest') = 1                
        BEGIN                
            INSERT INTO @configuraciones                
            select nId_Sede     as sCodigo,                
                   sDescripcion as sDescripcion,                
                   'SEDES_SUGGEST'      as sTipo_Configuracion                
            from Sedes         
   where sEntity = 'Empresa'  
   AND nEstado = 1 AND isSede_Usuarias = 1  
        END
```
