```SQL 
CREATE VIEW v_Listado_Tardanzas_Cierre_Report

AS

SELECT

vca.nId_Cierre,

vca.nId_Colaborador,

vca.sNombre_Colaborador,

c_doc.sDescripcion AS sTipo_Documento,

d.sNumero_Documento,

vca.dCantidad_Horas_Laboradas,

ISNULL(vca.dTardanzas_Con_Tolerancia_Horas,0) AS dTardanzas_Con_Tolerancia_Horas,

ISNULL(vca.dTardanzas_Sin_Tolerancia_Horas,0) AS dTardanzas_Sin_Tolerancia_Horas,

ISNULL(vca.dTiempo_Tardanza_Horas,0) AS dTiempo_Tardanza_Horas,

ISNULL(vca.dTiempo_Tardanza_Descontado,0) AS dTiempo_Tardanza_Descontado,

car.dFecha_Inicio_Tardanza,

car.dFecha_Fin_Tardanza,

ISNULL((SELECT COUNT(*) FROM Asistencias a

WHERE a.dFecha_Asistencia BETWEEN CONVERT(DATE, car.dFecha_Inicio_Tardanza) AND CONVERT(DATE, car.dFecha_Fin_Tardanza)

AND a.nEstado_Asistencia = 2 --PUNTUAL

AND a.nId_Colaborador = vca.nId_Colaborador

GROUP BY a.nId_Colaborador, a.nEstado_Asistencia), 0) AS nDias_Puntual,

ISNULL((SELECT COUNT(*) FROM Asistencias a

WHERE a.dFecha_Asistencia BETWEEN CONVERT(DATE, car.dFecha_Inicio_Tardanza) AND CONVERT(DATE, car.dFecha_Fin_Tardanza)

AND a.nEstado_Asistencia NOT IN(1,6,8,7,10,11)

AND a.nId_Colaborador = vca.nId_Colaborador

GROUP BY a.nId_Colaborador), 0) AS nDias_Laborales,

ISNULL(recu.nDias_Tardanza_Recuperable,0) AS nDias_Tardanza_Recuperable,

ISNULL(norecu.nDias_Tardanza_NO_Recuperable,0) AS nDias_Tardanza_NO_Recuperable

from cierreAsistencias vca

join cierreAsistenciaReportes car on vca.nId_Cierre = car.nId_Cierre

LEFT JOIN Colaboradores c ON c.nId_Colaborador = vca.nId_Colaborador

LEFT JOIN Personas p ON p.nId_Persona = c.nId_Persona

LEFT JOIN Documentos d ON p.nId_Persona = d.nId_Persona

LEFT JOIN Configs c_doc ON d.nTipo_Documento = c_doc.sCodigo

AND c_doc.sTabla = 'TIPO_DOCUMENTO'

LEFT JOIN (SELECT COUNT(*) AS nDias_Tardanza_Recuperable, nId_Colaborador, nId_Cierre FROM Transacciones_Saldo_Mins tsm

WHERE nId_Tipo_Entidad = 2 --TARDANZAS

AND nEstado = 1

AND nId_Sub_Tipo_Entidad = 3 -- RECUPERABLE

AND nId_Cierre IS NOT NULL

GROUP BY nId_Colaborador, nId_Cierre) recu ON recu.nId_Colaborador = vca.nId_Colaborador AND recu.nId_Cierre = vca.nId_Cierre

LEFT JOIN (SELECT COUNT(*) AS nDias_Tardanza_NO_Recuperable, nId_Colaborador, nId_Cierre FROM Transacciones_Saldo_Mins tsm

WHERE nId_Tipo_Entidad = 2 --TARDANZAS

AND nEstado = 1

AND nId_Sub_Tipo_Entidad = 4 -- NO RECUPERABLE

AND nId_Cierre IS NOT NULL

GROUP BY nId_Colaborador, nId_Cierre) norecu ON norecu.nId_Colaborador = vca.nId_Colaborador AND norecu.nId_Cierre = vca.nId_Cierre

WHERE (vca.dTardanzas_Sin_Tolerancia_Horas > 0 OR vca.dTardanzas_Con_Tolerancia_Horas > 0);
