```SQL
CREATE VIEW v_Listado_TareoStatus

AS

SELECT

t.nId_Colaborador,

p.sPersona_Nombre AS sNombre_Colaborador,

t.dFecha_Registro,

t.dDatetime_Creador,

DATEDIFF(DAY, t.dFecha_Registro, CAST(t.dDatetime_Creador AS DATE)) AS nDias_Diferencia,

dbo.fn_GetNextBusinessDay(t.dFecha_Registro) AS dSiguiente_DiaHabil,

t.nEstado,

t.nMinutos

FROM

Tareas t

LEFT JOIN Colaboradores c ON t.nId_Colaborador = c.nId_Colaborador

LEFT JOIN Personas p ON c.nId_Persona = p.nId_Persona

WHERE

t.nEstado = 1;