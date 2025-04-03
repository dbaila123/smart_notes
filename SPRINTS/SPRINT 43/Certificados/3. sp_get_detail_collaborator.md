```sql
CREATE   PROCEDURE sp_get_detail_collaborator    
(     
 @nId_Collaborator INT,      
 @nId_Person INT     
 )    
 AS    
SELECT nId_Colaborador, nEps, nTipo_Contratacion, dFec_Ingreso, nId_Empresa, nId_Area, nTipo_Colaborador, nTipo_Modalidad FROM Colaboradores WHERE nId_Colaborador = @nId_Collaborator;    
    
SELECT p.nId_Persona, p.sPersona_Nombre as sNombre_Completo, p.sPrimer_Nombre, p.sSegundo_Nombre, p.sApe_Paterno, p.sApe_Materno, p.dFecha_Nacimiento, p.nId_Sexo as sCodigo_Genero, p.nId_Nacionalidad as sCodigo_Nacionalidad, p.nId_Tipo_Persona as sCodigo_Tipo_Persona, p.nId_Estado_Civil as sCodigo_Estado_Civil FROM Personas p     
WHERE p.nId_Persona = @nId_Person;    
    
SELECT s.nId_Sede as sCodigo FROM Sedes s JOIN Sedes_Colaboradores sc ON sc.nId_Sede = s.nId_Sede     
WHERE sc.nId_Colaborador = @nId_Collaborator;    
    
SELECT c.nId_Cargo as sCodigo FROM Cargos c JOIN Cargos_Colaboradores cc ON cc.nId_Cargo = c.nId_Cargo     
WHERE cc.nId_Colaborador = @nId_Collaborator;    
    
SELECT p.nId_Proyecto as sCodigo, p.sNombre as sDescripcion, cp.nPrincipal as bPrincipal FROM Proyectos p JOIN Colaboradores_Proyectos cp ON p.nId_Proyecto = cp.nId_Proyecto     
WHERE cp.nId_Colaborador = @nId_Collaborator and cp.nEstado  = 1;    
    
    
SELECT     
   t.nId_Supervisor as sCodigo,     
t.bPrincipal,    
ISNULL(p.sPersona_Nombre, 'AUTOGESTIONADO') as sDescripcion    
FROM colaboradores_supervisor t     
LEFT JOIN Colaboradores c ON c.nId_Colaborador = t.nId_Supervisor     
LEFT JOIN Personas p ON c.nId_Persona = p.nId_Persona     
WHERE t.nId_Colaborador = @nId_Collaborator;    
    
SELECT hc.nId_Horario FROM Horarios_Colaboradores hc where hc.nId_Colaborador = @nId_Collaborator AND hc.nEstado = 1;     
    
SELECT d.nTipo_Documento as sCodigo_Tipo_Documento, d.sNumero_Documento FROM Documentos d WHERE d.nPrincipal = 1 AND d.nId_Persona = @nId_Person;    
    
SELECT dh.nId_Parentesco as sCodigo_Parentesco, p.sPersona_Nombre as sNombre_Completo, p.sPrimer_Nombre, p.sSegundo_Nombre, p.sApe_Paterno, p.sApe_Materno, p.dFecha_Nacimiento, p.nId_Sexo as sCodigo_Genero, p.nId_Nacionalidad as sCodigo_Nacionalidad, p.nId_Tipo_Persona as sCodigo_Tipo_Persona, p.nId_Estado_Civil as sCodigo_Estado_Civil,    
t.nTipo_Telefono as sCodigo_Tipo_Telefono, t.nId_Pais as sCodigo_Pais, t.sPrefijo_Area_Local, t.sTelefono     
FROM DerechoHabientes dh     
JOIN Personas p ON p.nId_Persona = dh.nId_Persona     
JOIN Telefonos t ON t.nId_Persona = dh.nId_Persona     
WHERE dh.nId_Colaborador = @nId_Collaborator;      
    
SELECT t.nTipo_Telefono as sCodigo_Tipo_Telefono, t.nId_Pais as sCodigo_Pais, t.sPrefijo_Area_Local, t.sTelefono  FROM Telefonos t WHERE t.nId_Persona = @nId_Person;    
    
SELECT e.nTipo_Email as sCodigo_Tipo_Email, e.sEmail, e.nPrincipal FROM Emails e WHERE e.nId_Persona = @nId_Person AND nTipo_Email = 1;    
     
SELECT d.nId_Ubigeo as sCodigo_Ubigeo, u.sUbigeo_Departamento as sDepartamento, u.sUbigeo_Provincia as sProvincia, u.sUbigeo_Distrito as sDistrito, d.sCalle as sDireccion, d.sCalle2 as sReferencia     
FROM Direcciones d     
JOIN Ubigeos u ON u.nId_Ubigeo = d.nId_Ubigeo     
WHERE d.nId_Persona = @nId_Person AND d.nPrincipal = 1;      
    
SELECT c.nParentesco as sCodigo_Parentesco, p.sPersona_Nombre as sNombre_Completo, p.sPrimer_Nombre, p.sSegundo_Nombre, p.sApe_Paterno, p.sApe_Materno, p.dFecha_Nacimiento, p.nId_Sexo as sCodigo_Genero, p.nId_Nacionalidad as sCodigo_Nacionalidad, p.nId_Tipo_Persona as sCodigo_Tipo_Persona, p.nId_Estado_Civil as sCodigo_Estado_Civil,    
t.nTipo_Telefono as sCodigo_Tipo_Telefono, t.nId_Pais as sCodigo_Pais, t.sPrefijo_Area_Local, t.sTelefono     
FROM Contactos c    
JOIN Personas p ON p.nId_Persona = c.nId_Persona_Contacto     
JOIN Telefonos t ON t.nId_Persona = c.nId_Persona_Contacto     
WHERE c.nId_Persona = @nId_Person AND c.nId_Persona_Contacto NOT IN (SELECT dh.nId_Persona FROM DerechoHabientes dh WHERE DH.nId_Colaborador = @nId_Collaborator);    
    
SELECT cb.nId_Banco as sCodigo_Entidad_Bancaria, cb.nTipo_Cuenta as sCodigo_Tipo_Cuenta, cb.nNumero_Cuenta as sNumero_Cuenta, cb.sCCI FROM Cuentas_Bancarias cb WHERE cb.nId_Persona = @nId_Person;    
    
SELECT u.nId_Usuario, u.nTipo_Usuario, u.nTipo_Autenticacion, u.sClave, u.sEmail, ru.nId_Rol     
FROM Usuarios u     
JOIN Roles_Usuarios ru on ru.nId_Usuario = u.nId_Usuario     
WHERE u.nId_Persona = @nId_Person AND ru.nEstado = 1;     
    
SELECT     
    cc.nId_Certificado,    
    cc.nId_Colaborador,    
    cc.sNombre,    
    cc.sUrl,    
    cc.dFecha_Inicio,    
    cc.dFecha_Expiracion,    
    cc.nCantidad_Horas,    
    cc.sDescripcion,   
    (    
        SELECT STRING_AGG(f.sUrl_File, '*')    
        FROM Files f    
        WHERE f.nId_Entidad = cc.nId_Certificado     
        AND f.sEntidad = 'Certificado_Colaborador'    
        AND f.nEstado = 1    
    ) AS sUrl_Files,  
 cc.nEstado  
FROM     
    collaborators_certificate cc    
WHERE    
 cc.nId_Colaborador = @nId_Collaborator;
```
