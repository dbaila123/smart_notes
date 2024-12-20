```SQL
CREATE VIEW v_Listado_Tardanzas_Cierre

AS

SELECT DISTINCT

tsm.nId_Colaborador,

p.sPersona_Nombre AS sNombre_Colaborador,

CONCAT(CASE

WHEN c_doc.sDescripcion = 'PASAPORTE' THEN 'PAS'

ELSE c_doc.sDescripcion

END, ' ', d.sNumero_Documento) AS sDocumento_Identidad,

c.sUrl_Foto AS sUrl_Image,

tsm.dTardanzas_Con_Tolerancia_Horas,

tsm.dTardanzas_Sin_Tolerancia_Horas,

tsm.dTiempo_Tardanza_Horas,

ISNULL((CASE

WHEN cap.nEstado_Cierre = 3 -- CERRADO

THEN tsm.dTiempo_Tardanza_Descontado

ELSE

(SELECT SUM(dTiempo_Tardanza)

FROM Asistencias a

WHERE a.nId_Colaborador = tsm.nId_Colaborador

AND a.dFecha_Asistencia

BETWEEN cap.dFecha_Inicio_Tardanza AND cap.dFecha_Fin_Tardanza

AND a.bDescontar_tardanza = 1

GROUP BY a.nId_Colaborador)

END

),0) AS dTiempo_Tardanza_Descontado,

(

SELECT

a.dTiempo_Tardanza,

CONVERT(varchar(23), a.dFecha_Asistencia, 126) AS dFecha_Asistencia,

ISNULL(a.bDescontar_tardanza, 0) AS bDescontar_tardanza,

a.nId_Asistencia

FROM Asistencias a

WHERE a.nId_Colaborador = tsm.nId_Colaborador

AND a.dFecha_Asistencia BETWEEN cap.dFecha_Inicio_Tardanza AND cap.dFecha_Fin_Tardanza

FOR JSON PATH

) AS sTardanzas_x_Dias,

tsm.nId_Cierre,

(SELECT nId_Supervisor FROM fn_supervisorsName_by_collaborator(tsm.nId_Colaborador) fn WHERE fn.nIsFinalApprover = 1) AS nId_Director

FROM cierreAsistencias tsm

JOIN Colaboradores c ON tsm.nId_Colaborador = c.nId_Colaborador

JOIN Personas p ON c.nId_Persona = p.nId_Persona

LEFT JOIN Documentos d ON p.nId_Persona = d.nId_Persona AND d.nPrincipal = 1

LEFT JOIN Configs c_doc ON d.nTipo_Documento = c_doc.sCodigo AND c_doc.sTabla = 'TIPO_DOCUMENTO'

JOIN cierreAsistenciaReportes cap ON cap.nId_Cierre = tsm.nId_Cierre

WHERE dTiempo_Tardanza_Horas > 0;
```