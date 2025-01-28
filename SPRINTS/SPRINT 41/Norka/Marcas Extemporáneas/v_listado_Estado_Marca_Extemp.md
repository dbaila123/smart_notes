```SQL
CREATE VIEW dbo.v_Listado_Estado_Marca_Extemp

AS

SELECT

m.nId_Colaborador,

m.nId_Marca,

CASE

WHEN p.sPrimer_Nombre = '' OR p.sApe_Paterno = ''

THEN p.sPersona_Nombre

ELSE p.sPrimer_Nombre + ' ' + p.sApe_Paterno

END AS sPersona_Nombre,

m.dFecha_Jornada,

dbo.fg_get_calcular_estado_marca_extemp(m.nId_Colaborador, m.dFecha_Jornada) AS ESTADO_MARCA

FROM

MARCAS m

JOIN

COLABORADORES c ON c.nId_Colaborador = m.nId_Colaborador

JOIN

PERSONAS p ON c.nId_Persona = p.nId_Persona;
```