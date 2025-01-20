```SQL
-- dbo.v_Listado_Asistencias_Reporte source

  

CREATE OR ALTER VIEW [dbo].[v_Listado_Asistencias_Reporte] AS

SELECT

a.nId_Asistencia AS nId,

a.nId_Colaborador,

a.nId_Modalidad AS nTipo_Modalidad,

c3.sDescripcion AS sTipo_Modalidad,

a.dFecha_Asistencia AS dFecha_Registro_Asistencia,

DATEPART ( month , a.dFecha_Asistencia ) AS nMes,

DATEPART ( year , a.dFecha_Asistencia ) AS nAnio,

a.nEstado_Asistencia AS nEstado,

a.dTiempo_Tardanza AS dCantidad_Tardanza,

c2.sDescripcion AS sEstado,

p.sPersona_Nombre AS sNombre_Colaborador,

u.nId_Usuario AS nId_User,

--t2.nId_Team,

--u_director.nId_Usuario AS nId_User_Director,

--u_lider.nId_Usuario AS nId_User_Lider,

a.dTiempo_Laboradas AS nTotal_Horas_Trabajadas,

c_doc.sDescripcion as sTipo_Documento,

d.sNumero_Documento,

CASE

WHEN DATEPART(dw, a.dFecha_Asistencia) = 6 THEN 'Viernes'

WHEN DATEPART(dw, a.dFecha_Asistencia) = 5 THEN 'Jueves'

WHEN DATEPART(dw, a.dFecha_Asistencia) = 4 THEN 'Miércoles'

WHEN DATEPART(dw, a.dFecha_Asistencia) = 3 THEN 'Martes'

WHEN DATEPART(dw, a.dFecha_Asistencia) = 2 THEN 'Lunes'

WHEN DATEPART(dw, a.dFecha_Asistencia) = 1 THEN 'Domingo'

WHEN DATEPART(dw, a.dFecha_Asistencia) = 7 THEN 'Sábado'

END sDia,

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

-- mEntrada.sJustificacion sJustificacion,

cast(concat_ws(' ',

iif(mEntrada.sJustificacion is not null, mEntrada.sComentario + ': ', ''),

mEntrada.sJustificacion + '- ',

iif(mSalida_Almuerzo.sJustificacion is not null, mSalida_Almuerzo.sComentario + ': ', ''),

mSalida_Almuerzo.sJustificacion + '- ',

iif(mIngreso_Almuerzo.sJustificacion is not null, mIngreso_Almuerzo.sComentario + ': ', ''),

mIngreso_Almuerzo.sJustificacion + '- ',

iif(mSalida.sJustificacion is not null, mSalida.sComentario + ': ', ''),

mSalida.sJustificacion + '- '

) as varchar(max)) sJustificacion,

mSalida.Hora_Salida,

mSalida_Almuerzo.Hora_Salida_Almuerzo,

mIngreso_Almuerzo.Hora_Ingreso_Almuerzo,

CASE

WHEN mEntrada.nEstadoMarca = 0 THEN 1

ELSE null

END nEstado_Solicitud_Marca_Entrada,

CASE

WHEN mSalida.nEstadoMarca = 0 THEN 1

ELSE null

END nEstado_Solicitud_Marca_Salida,

(SELECT

STRING_AGG(

CASE

WHEN ts.nTipo_Frecuencia = 1 THEN

CONCAT(ts.sDescripcion, ' DE ', LEFT(CAST(s.dHora_Inicio AS NVARCHAR(20)), 5), ' A ', LEFT(CAST(s.dHora_Fin AS NVARCHAR(20)), 5))

ELSE

CONCAT(ts.sDescripcion, ' DE ', s.dFecha_Inicio, ' A ', s.dFecha_Fin)

END, ', ') AS sObservacion

FROM

Solicitudes s

JOIN Tipos_Solicitudes ts ON s.nId_Tipo_Solicitud = ts.nId_Tipo_Solicitud

WHERE

(a.dFecha_Asistencia BETWEEN s.dFecha_Inicio AND s.dFecha_Fin) AND

s.nEstado_Solicitud IN (3) AND

ts.nId_Categoria NOT IN (4) AND

s.nId_Colaborador = a.nId_Colaborador) AS sObservacion

FROM Asistencias a

JOIN Colaboradores c ON c.nId_Colaborador = a.nId_Colaborador

JOIN Personas p ON p.nId_Persona = c.nId_Persona

LEFT JOIN Usuarios u ON u.nId_Persona = p.nId_Persona

LEFT JOIN Documentos d ON p.nId_Persona = d.nId_Persona

LEFT JOIN Configs c_doc ON d.nTipo_Documento = c_doc.sCodigo

AND c_doc.sTabla = 'TIPO_DOCUMENTO'

--LEFT JOIN Team_Colaboradors tc ON tc.nId_Colaborador = a.nId_Colaborador AND tc.nPrincipal = 1

--JOIN Team t2 ON t2.nId_Team = tc.nId_Team

--JOIN Colaboradores c_lider ON t2.nId_Lider = c_lider.nId_Colaborador

--JOIN Usuarios u_lider ON u_lider.nId_Persona = c_lider.nId_Persona

--JOIN Colaboradores c_director ON t2.nId_Director = c_director.nId_Colaborador

--JOIN Usuarios u_director ON u_director.nId_Persona = c_director.nId_Persona

LEFT JOIN (

-- 49,215

SELECT m.nId_Colaborador, m.dFecha_Jornada, m.nEstado AS nEstadoMarca, CONVERT(TIME, m.dFecha_Marca, 108) AS Hora_Ingreso, m.sJustificacion, m.nMarcaje_Extemporaneo, c.sComentario

FROM Marcas m

left join Configs c on c.sTabla = 'TIPO_MARCACION' and c.sCodigo = cast(m.nTipo_Marcacion as varchar(5))

WHERE m.nTipo_Marcacion = 1

AND (m.nEstado = 1 OR (m.nMarcaje_Extemporaneo = 1 and m.nEstado = 0))

GROUP BY m.nId_Colaborador, m.dFecha_Jornada, m.nEstado, m.dFecha_Marca, m.sJustificacion, m.nMarcaje_Extemporaneo, c.sComentario

) mEntrada ON a.dFecha_Asistencia = CONVERT(DATE, mEntrada.dFecha_Jornada)

AND mEntrada.nId_Colaborador = c.nId_Colaborador

LEFT JOIN (

SELECT m.nId_Colaborador, m.dFecha_Jornada, m.nEstado AS nEstadoMarca, CONVERT(TIME, m.dFecha_Marca, 108) AS Hora_Salida, m.nMarcaje_Extemporaneo, min(m.sJustificacion) sJustificacion, c.sComentario

FROM Marcas m

left join Configs c on c.sTabla = 'TIPO_MARCACION' and c.sCodigo = cast(m.nTipo_Marcacion as varchar(5))

WHERE m.nTipo_Marcacion = 4

AND (m.nEstado = 1 OR (nMarcaje_Extemporaneo = 1 and m.nEstado = 0))

GROUP BY m.nId_Colaborador, m.dFecha_Jornada, m.nEstado, m.dFecha_Marca, m.nMarcaje_Extemporaneo, c.sComentario

) mSalida ON a.dFecha_Asistencia = CONVERT(DATE, mSalida.dFecha_Jornada)

AND mSalida.nId_Colaborador = c.nId_Colaborador

LEFT JOIN (

SELECT m.nId_Colaborador, m.dFecha_Jornada, m.nEstado AS nEstadoMarca, CONVERT(TIME, m.dFecha_Marca, 108) AS Hora_Salida_Almuerzo, m.nMarcaje_Extemporaneo, min(m.sJustificacion) sJustificacion, c.sComentario

FROM Marcas m

left join Configs c on c.sTabla = 'TIPO_MARCACION' and c.sCodigo = cast(m.nTipo_Marcacion as varchar(5))

WHERE m.nTipo_Marcacion = 2

AND (m.nEstado = 1 OR (m.nMarcaje_Extemporaneo = 1 and m.nEstado = 0))

GROUP BY m.nId_Colaborador, m.dFecha_Jornada, m.nEstado, m.dFecha_Marca, m.nMarcaje_Extemporaneo, c.sComentario

) mSalida_Almuerzo ON a.dFecha_Asistencia = CONVERT(DATE, mSalida_Almuerzo.dFecha_Jornada)

AND mSalida_Almuerzo.nId_Colaborador = c.nId_Colaborador

LEFT JOIN (

SELECT m.nId_Colaborador, m.dFecha_Jornada, m.nEstado AS nEstadoMarca, CONVERT(TIME, m.dFecha_Marca, 108) AS Hora_Ingreso_Almuerzo, m.nMarcaje_Extemporaneo, min(m.sJustificacion) sJustificacion, c.sComentario

FROM Marcas m

left join Configs c on c.sTabla = 'TIPO_MARCACION' and c.sCodigo = cast(m.nTipo_Marcacion as varchar(5))

WHERE m.nTipo_Marcacion = 3

AND (m.nEstado = 1 OR (m.nMarcaje_Extemporaneo = 1 and m.nEstado = 0))

GROUP BY m.nId_Colaborador, m.dFecha_Jornada, m.nEstado, m.dFecha_Marca, m.nMarcaje_Extemporaneo, c.sComentario

) mIngreso_Almuerzo ON a.dFecha_Asistencia = CONVERT(DATE, mIngreso_Almuerzo.dFecha_Jornada)

AND mIngreso_Almuerzo.nId_Colaborador = c.nId_Colaborador

JOIN Configs c2 ON c2.sCodigo = CONVERT(nvarchar(max),a.nEstado_Asistencia) AND c2.sTabla = 'ESTADO_ASISTENCIA'

JOIN Configs c3 ON c3.sCodigo = CONVERT(nvarchar(max),a.nId_Modalidad) AND c3.sTabla = 'TIPO_MODALIDAD';
```
