```SQL
CREATE OR ALTER VIEW v_Listado_TareoStatus

AS

SELECT

t.nId_Colaborador,

CONCAT(p.sPrimer_Nombre, ' ', p.sApe_Paterno) AS sNombre_Colaborador,

p.sPersona_Nombre AS sNombre_Completo,

t.dFecha_Registro,

t.dDatetime_Creador,

DATEDIFF(DAY, t.dFecha_Registro, CAST(t.dDatetime_Creador AS DATE)) AS nDias_Diferencia,

dbo.fn_GetNextBusinessDay(t.dFecha_Registro) AS dSiguiente_DiaHabil,

t.nEstado,

t.nMinutos,

c.nEstado_Colaborador

FROM

Tareas t

INNER JOIN Colaboradores c

ON t.nId_Colaborador = c.nId_Colaborador

LEFT JOIN Personas p

ON c.nId_Persona = p.nId_Persona

WHERE

t.nEstado = 1;