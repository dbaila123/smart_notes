```SQL
CREATE OR ALTER VIEW [dbo].[v_Listado_Asistencias] AS

SELECT

a.nId_Asistencia AS nId,

a.nId_Colaborador,

a.nId_Modalidad AS nTipo_Modalidad,

c3.sDescripcion AS sTipo_Modalidad,

a.dFecha_Asistencia AS dFecha_Registro_Asistencia,

a.nEstado_Asistencia AS nEstado,

a.dTiempo_Tardanza AS dCantidad_Tardanza,

c2.sDescripcion AS sEstado,

p.sPrimer_Nombre,

p.sPersona_Nombre AS sNombre_Colaborador,

CONCAT(ISNULL(c_doc.sDescripcion,''), ' ', ISNULL(d.sNumero_Documento,'')) AS sDocumento_Identidad,

c_doc.sDescripcion as sTipo_Documento,

d.sNumero_Documento,

c.sUrl_Foto,

t.sTelefono AS sContacto_Telefonico,

a.nId_Sede,

s.sDescripcion AS sNombre_Sede,

u.nId_Usuario AS nId_User,

--t2.nId_Team,

a.nMinutos_Permiso,

--u_director.nId_Usuario AS nId_User_Director,

--u_lider.nId_Usuario AS nId_User_Lider,

vhdth.[S. Almuerzo] AS hora_Salida_Almuerzo_Horario,

vhdth.[R. Almuerzo] AS hora_Ingreso_Almuerzo_Horario,

DATEDIFF(MINUTE, CONVERT(DATETIME, vhdth.[S. Almuerzo], 121), -- Assuming format YYYY-MM-DD HH:MM:SS

CONVERT(DATETIME, vhdth.[R. Almuerzo], 121)) AS nMinutos_Almuerzo_horario,

DATEDIFF(MINUTE, CONVERT(DATETIME, vhdth.Entrada, 121), -- Assuming format YYYY-MM-DD HH:MM:SS

CONVERT(DATETIME, vhdth.Salida, 121)) AS nMinutos_Jornada_Horario,

CASE

WHEN ah.cnt > 0 THEN 1

ELSE 0

END FlagHistorico,

CASE

WHEN (SELECT COUNT(*) FROM Solicitudes s2

JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s2.nId_Tipo_Solicitud

WHERE s2.nId_Colaborador = a.nId_Colaborador

AND a.dFecha_Asistencia BETWEEN s2.dFecha_Inicio and s2.dFecha_Fin

AND s2.nId_Tipo_Solicitud NOT IN (5,6,7,8,9,10,20,21)

AND s2.nEstado_Solicitud NOT IN (2,6,7)) > 0

THEN 1

ELSE 0

END bPermiso,

mEntrada.Hora_Ingreso,

mEntrada.sJustificacion sJustificacion ,

iif(mEntrada.sJustificacion is null, 0, 1) bJustificacion_Hora_Ingreso,

mSalida.Hora_Salida,

iif(mSalida.sJustificacion is null, 0, 1) bJustificacion_Hora_Salida,

mSalida_Almuerzo.Hora_Salida_Almuerzo,

iif(mSalida_Almuerzo.sJustificacion is null, 0, 1) bJustificacion_Hora_Salida_Almuerzo,

mIngreso_Almuerzo.Hora_Ingreso_Almuerzo,

iif(mIngreso_Almuerzo.sJustificacion is null, 0, 1) bJustificacion_Hora_Ingreso_Almuerzo,

DATEDIFF(MINUTE, CONVERT(DATETIME, LEFT(mSalida_Almuerzo.Hora_Salida_Almuerzo, 8), 121),

CONVERT(DATETIME, LEFT(mIngreso_Almuerzo.Hora_Ingreso_Almuerzo, 8), 121)) AS nMinutos_Almuerzo,

a.dTiempo_Laboradas AS nMinutos_Jornada,

CASE

WHEN mEntrada.nEstadoMarca = 0 THEN 1

ELSE null

END nEstado_Solicitud_Marca_Entrada,

CASE

WHEN mSalida.nEstadoMarca = 0 THEN 1

ELSE null

END nEstado_Solicitud_Marca_Salida,

  

-- obtener coordenadas de marcaciones

mEntrada.nLatitude EntradaLatitude,

mEntrada.nLongitude EntradaLongitude,

  

mSalida.nLatitude SalidaLatitude,

mSalida.nLongitude SalidadLongitude,

  

mSalida_Almuerzo.nLatitude Salida_AlmuerzoLatitude,

mSalida_Almuerzo.nLongitude Salida_AlmuerzoLongitude,

  

mIngreso_Almuerzo.nLatitude Ingreso_AlmuerzoLatitude,

mIngreso_Almuerzo.nLongitude Ingreso_AlmuerzoLongitude

  

FROM Asistencias a

LEFT JOIN Colaboradores c ON c.nId_Colaborador = a.nId_Colaborador

LEFT JOIN Personas p ON p.nId_Persona = c.nId_Persona

LEFT JOIN Documentos d ON p.nId_Persona = d.nId_Persona

LEFT JOIN Configs c_doc ON d.nTipo_Documento = c_doc.sCodigo

AND c_doc.sTabla = 'TIPO_DOCUMENTO'

LEFT JOIN Usuarios u ON u.nId_Persona = p.nId_Persona

--LEFT JOIN Team_Colaboradors tc ON tc.nId_Colaborador = a.nId_Colaborador AND tc.nPrincipal = 1

--JOIN Team t2 ON t2.nId_Team = tc.nId_Team

--JOIN Colaboradores c_lider ON t2.nId_Lider = c_lider.nId_Colaborador

--JOIN Usuarios u_lider ON u_lider.nId_Persona = c_lider.nId_Persona

--JOIN Colaboradores c_director ON t2.nId_Director = c_director.nId_Colaborador

--JOIN Usuarios u_director ON u_director.nId_Persona = c_director.nId_Persona

LEFT JOIN Telefonos t ON p.nId_Persona = t.nId_Persona AND t.nPrincipal = 1

JOIN Sedes s ON a.nId_Sede = s.nId_Sede

--LEFT JOIN Horarios_Colaboradores hc ON hc.nId_Horario = a.nId_Horario --AND hc.nEstado = 1

LEFT JOIN v_horario_dias_total_horas vhdth ON vhdth.nId_Horario = a.nId_Horario AND vhdth.nDia = DATEPART(WEEKDAY, a.dFecha_Asistencia) - 1

LEFT JOIN (

SELECT nId_Colaborador, dFecha_Jornada, nEstado AS nEstadoMarca, CONVERT(DATE, dFecha_Marca) AS fecha_Marca, CONVERT(TIME, dFecha_Marca, 108) AS Hora_Ingreso, sJustificacion, nMarcaje_Extemporaneo, nLatitude, nLongitude

FROM Marcas

WHERE nTipo_Marcacion = 1

AND (nEstado = 1 OR (nMarcaje_Extemporaneo = 1 and nEstado = 0))

GROUP BY nId_Colaborador, dFecha_Jornada, nEstado, dFecha_Marca, sJustificacion, nMarcaje_Extemporaneo, nLatitude, nLongitude

) mEntrada ON a.dFecha_Asistencia = CONVERT(DATE, mEntrada.dFecha_Jornada)

AND mEntrada.nId_Colaborador = c.nId_Colaborador

LEFT JOIN (

SELECT nId_Colaborador, dFecha_Jornada, nEstado AS nEstadoMarca, CONVERT(DATE, dFecha_Marca) AS fecha_Marca , CONVERT(TIME, dFecha_Marca, 108) AS Hora_Salida, min(sJustificacion) sJustificacion, nMarcaje_Extemporaneo, nLatitude, nLongitude

FROM Marcas

WHERE nTipo_Marcacion = 4

AND (nEstado = 1 OR (nMarcaje_Extemporaneo = 1 and nEstado = 0))

GROUP BY nId_Colaborador, dFecha_Jornada, nEstado, dFecha_Marca, nMarcaje_Extemporaneo, nLatitude, nLongitude

) mSalida ON a.dFecha_Asistencia = CONVERT(DATE, mSalida.dFecha_Jornada)

AND mSalida.nId_Colaborador = c.nId_Colaborador

LEFT JOIN (

SELECT nId_Colaborador, dFecha_Jornada, nEstado AS nEstadoMarca, CONVERT(TIME, dFecha_Marca, 108) AS Hora_Salida_Almuerzo, min(sJustificacion) sJustificacion, nMarcaje_Extemporaneo, nLatitude, nLongitude

FROM Marcas

WHERE nTipo_Marcacion = 2

AND (nEstado = 1 OR (nMarcaje_Extemporaneo = 1 and nEstado = 0))

GROUP BY nId_Colaborador, dFecha_Jornada, nEstado, dFecha_Marca, nMarcaje_Extemporaneo, nLatitude, nLongitude

) mSalida_Almuerzo ON a.dFecha_Asistencia = CONVERT(DATE, mSalida_Almuerzo.dFecha_Jornada)

AND mSalida_Almuerzo.nId_Colaborador = c.nId_Colaborador

LEFT JOIN (

SELECT nId_Colaborador, dFecha_Jornada, nEstado AS nEstadoMarca, CONVERT(TIME, dFecha_Marca, 108) AS Hora_Ingreso_Almuerzo, min(sJustificacion) sJustificacion, nMarcaje_Extemporaneo, nLatitude, nLongitude

FROM Marcas

WHERE nTipo_Marcacion = 3

AND (nEstado = 1 OR (nMarcaje_Extemporaneo = 1 and nEstado = 0))

GROUP BY nId_Colaborador, dFecha_Jornada, nEstado, dFecha_Marca, nMarcaje_Extemporaneo, nLatitude, nLongitude

) mIngreso_Almuerzo ON a.dFecha_Asistencia = CONVERT(DATE, mIngreso_Almuerzo.dFecha_Jornada)

AND mIngreso_Almuerzo.nId_Colaborador = c.nId_Colaborador

LEFT JOIN (

SELECT nId_Asistencia, COUNT(*) AS cnt

FROM Asistencia_Historicos

GROUP BY nId_Asistencia

) ah ON a.nId_Asistencia = ah.nId_Asistencia

JOIN Configs c2 ON c2.sCodigo = CONVERT(nvarchar(max),a.nEstado_Asistencia) AND c2.sTabla = 'ESTADO_ASISTENCIA'

JOIN Configs c3 ON c3.sCodigo = CONVERT(nvarchar(max),a.nId_Modalidad) AND c3.sTabla = 'TIPO_MODALIDAD';
```