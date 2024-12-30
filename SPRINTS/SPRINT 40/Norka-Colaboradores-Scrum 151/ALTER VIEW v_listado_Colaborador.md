```SQL
ALTER VIEW v_listado_Colaborador

AS

SELECT

c.nId_Colaborador AS nId,

p.sPersona_Nombre AS sNombre_Colaborador,

d.nTipo_Documento,

CONCAT(

CASE

WHEN c_doc.sDescripcion = 'PASAPORTE' THEN 'PAS'

ELSE c_doc.sDescripcion

END,

' ',

d.sNumero_Documento

) AS sDocumento_Identidad,

CONVERT(DATETIME2(7), c.dDatetime_Creador) AS dFec_Registro,

CONVERT(DATETIME2(7), c.dFec_Ingreso) AS dFec_Ingreso,

CONVERT(DATETIME2(7), p.dFecha_Nacimiento) AS dFecha_Nacimiento,

t.sTelefono AS nTelefono,

c.nTipo_Modalidad,

s.sDescripcion AS sNombre_Sede,

s.nId_Sede AS nId_Sede,

C6.sNombre AS sNombre_Cargo,

C6.nId_Cargo AS nId_Cargo,

p2.sPersona_Nombre AS sNombre_Empresa,

eu.nId_Empresa AS nId_Empresa,

c.nEstado_Colaborador AS nEstado,

c2.sDescripcion AS sNombre_Estado,

c3.sDescripcion AS sNombre_Modalidad,

CONVERT(DATETIME2(7), c.dDatetime_Delete) AS dFecha_Cese,

c.sUrl_Foto,

(

SELECT

CONCAT(percol.sPrimer_Nombre, ' ', percol.sApe_paterno) AS sNombre_Supervisor,

supcol.nId_Supervisor,

supcol.nOrden

FROM fb_get_supervisors_by_collaborators(c.nId_Colaborador, NULL) supcol

JOIN Colaboradores supcolinfo ON supcol.nId_Supervisor = supcolinfo.nId_Colaborador

JOIN Personas percol ON percol.nId_persona = supcolinfo.nId_Persona

FOR JSON PATH

) AS SupervisoresJson

FROM Colaboradores c

LEFT JOIN Personas p ON p.nId_Persona = c.nId_Persona

LEFT JOIN Documentos d ON d.nId_Persona = p.nId_Persona AND d.nPrincipal = 1

LEFT JOIN Configs c_doc ON d.nTipo_Documento = c_doc.sCodigo AND c_doc.sTabla = 'TIPO_DOCUMENTO'

LEFT JOIN Telefonos t ON t.nId_Persona = p.nId_Persona AND t.nPrincipal = 1

LEFT JOIN Sedes_Colaboradores sc ON sc.nId_Colaborador = c.nId_Colaborador

LEFT JOIN Sedes s ON s.nId_Sede = sc.nId_Sede

LEFT JOIN Cargos_Colaboradores cc ON cc.nId_Colaborador = c.nId_Colaborador

LEFT JOIN Cargos c6 on CC.nId_Cargo = C6.nId_Cargo

LEFT JOIN Empresas_Usuarias eu ON eu.nId_Empresa = c.nId_Empresa

LEFT JOIN Personas p2 ON p2.nId_Persona = eu.nId_Persona

LEFT JOIN Configs c2 ON c2.sCodigo = CONVERT(nvarchar(max),c.nEstado_Colaborador) AND c2.sTabla = 'ESTADO_COLABORADOR'

LEFT JOIN Configs c3 ON c3.sCodigo = CONVERT(nvarchar(max),c.nTipo_Modalidad) AND c3.sTabla = 'TIPO_MODALIDAD'

LEFT JOIN usuarios u ON p.nId_Persona = u.nId_Persona

LEFT JOIN Roles_Usuarios ru ON ru.nId_Usuario = u.nId_Usuario

WHERE ru.nId_Rol NOT IN (1, 15) OR u.nId_Usuario IS NULL;
```