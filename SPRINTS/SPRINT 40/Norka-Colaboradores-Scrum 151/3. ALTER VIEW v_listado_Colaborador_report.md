```SQL
ALTER VIEW v_listado_Colaborador_report

AS

SELECT

c.nId_Colaborador AS nId,

UPPER(p.sPersona_Nombre) AS sNombre_Colaborador,

con.sDescripcion as sTipo_Documento,

d.sNumero_Documento,

CONVERT(DATETIME2(7), c.dDatetime_Creador) AS dFec_Registro,

CONVERT(DATETIME2(7), c.dFec_Ingreso) AS dFec_Ingreso,

CONVERT(DATETIME2(7), p.dFecha_Nacimiento) AS dFecha_Nacimiento,

c3.sDescripcion AS sNombre_Modalidad,

c.nTipo_Modalidad,

c.nEstado_Colaborador AS nEstado,

s.nId_Sede AS nId_Sede,

s.sDescripcion AS sNombre_Sede,

C6.sDescripcion AS sNombre_Cargo,

C6.nId_Cargo AS nId_Cargo,

p2.sPersona_Nombre AS sNombre_Empresa,

eu.nId_Empresa AS nId_Empresa,

CONVERT(DATETIME2(7), c.dDatetime_Delete) AS dFecha_Cese,

c2.sDescripcion AS sNombre_Estado,

dbo.fg_get_supervisors_director_by_collaborators(c.nId_Colaborador, 1) AS sNombre_Director,

dbo.fg_get_supervisors_director_by_collaborators(c.nId_Colaborador, 0) AS sNombre_Lider,

cl.dFecha_Reingreso,

c.sMotivo_Cese

FROM Colaboradores c

LEFT JOIN Personas p ON p.nId_Persona = c.nId_Persona

LEFT JOIN Documentos d ON d.nId_Persona = p.nId_Persona AND d.nPrincipal = 1

LEFT JOIN Configs con ON d.nTipo_Documento = con.sCodigo AND con.sTabla = 'TIPO_DOCUMENTO'

LEFT JOIN Telefonos t ON t.nId_Persona = p.nId_Persona AND t.nPrincipal = 1

LEFT JOIN Sedes_Colaboradores sc ON sc.nId_Colaborador = c.nId_Colaborador

LEFT JOIN Sedes s ON s.nId_Sede = sc.nId_Sede

LEFT JOIN Cargos_Colaboradores cc ON cc.nId_Colaborador = c.nId_Colaborador

LEFT JOIN Cargos c6 ON CC.nId_Cargo = C6.nId_Cargo

LEFT JOIN Empresas_Usuarias eu ON eu.nId_Empresa = c.nId_Empresa

LEFT JOIN Personas p2 ON p2.nId_Persona = eu.nId_Persona

LEFT JOIN Configs c2 ON c2.sCodigo = CONVERT(nvarchar(max), c.nEstado_Colaborador) AND c2.sTabla = 'ESTADO_COLABORADOR'

LEFT JOIN Configs c3 ON c3.sCodigo = CONVERT(nvarchar(max), c.nTipo_Modalidad) AND c3.sTabla = 'TIPO_MODALIDAD'

LEFT JOIN Colaboradores_logs cl ON c.nId_Colaborador = cl.nId_Colaborador

LEFT JOIN usuarios u ON p.nId_Persona = u.nId_Persona

LEFT JOIN Roles_Usuarios ru ON ru.nId_Usuario = u.nId_Usuario

WHERE ru.nId_Rol NOT IN (1, 15) OR u.nId_Usuario IS NULL;
```