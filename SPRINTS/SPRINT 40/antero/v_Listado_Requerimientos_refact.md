```sql
-- dbo.v_Listado_Requerimientos_refact source

  

CREATE OR ALTER VIEW [dbo].[v_Listado_Requerimientos_refact]

AS

SELECT DISTINCT

r.nId_Requerimiento ,

CONCAT(r.sCodigo, ' ', r.sNombre) AS sNombre_Requerimiento,

--r.sNombre sNombre_Requerimiento,

p.nId_Proyecto,

r.nId_Lider as nId_Lider_Rq,

p.nId_Lider as nLiderPrincipal,

CONCAT(pLider.sPrimer_Nombre, ' ', pLider.sApe_Paterno) as sNombreLider,

CONCAT(pDirector.sPrimer_Nombre, ' ', pDirector.sApe_Paterno) as sNombreDirector,

p.sNombre sNombre_Proyecto,

CONVERT(DATE,r.dFecha_Inicio) dFecha_Inicio,

MIN(t2.dFecha_Registro) dFechaInicioRqReal,

CONVERT(DATE,r.dFecha_Fin) dFecha_Fin,

MAX(t2.dFecha_Registro) dFechaFinRqReal,

r.dFecha_Modificacion_Estado,

p_coordinador.sPersona_Nombre Nombre_Coordinador,

p2.sPersona_Nombre Nombre_Cliente,

r.nEstado nEstado_Requerimiento,

(select isnull(sum(nEstimacion),0) from Requerimientos_Categoria rc where nId_Requerimiento = r.nId_Requerimiento) nEstimacion,

co.sDescripcion sEstado_Requerimiento,

CASE WHEN

(select count(*) from justificaciones_Proyectos_Requerimientos jr where jr.nId_Entidad = r.nId_Requerimiento and jr.sEntidad = 'REQUERIMIENTO') > 0

THEN convert(bit,1)

ELSE convert(bit,0)

END bJustificacion,

r.dDatetime_Creador dFecha_Creacion,

CASE

WHEN r.nUsuario_Creador = 172 or r.nUsuario_Creador = 666

THEN 'SMART'

ELSE

(

SELECT TOP 1

CONCAT(tb_person_creator.sPrimer_Nombre, ' ', tb_person_creator.sApe_Paterno)

FROM Personas tb_person_creator

JOIN Usuarios tb_user_creator ON tb_user_creator.nId_Persona = tb_person_creator.nId_Persona

WHERE tb_user_creator.nId_Usuario = r.nUsuario_Creador

)

END sNombre_Usuario_Creador,

(SELECT c.sDescripcion from Configs c where sTabla = 'TIPO_SERVICIO' and c.sCodigo = p.sId_Tipo_Servicio ) as sTipo_Servicio_Req,

r.sCodigo as sCodigo

FROM Requerimientos r

JOIN Proyectos p ON p.nId_Proyecto = r.nId_Proyecto

JOIN Colaboradores cLider ON cLider.nId_Colaborador = r.nId_Lider

JOIN Personas pLider ON pLider.nId_Persona = cLider.nId_Persona

JOIN Colaboradores cDirector ON cDirector.nId_Colaborador = p.nId_Director

JOIN Personas pDirector ON pDirector.nId_Persona = cDirector.nId_Persona

JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto

JOIN Coordinadores c ON c.nId_Coordinador = cp.nId_Coordinador

JOIN Personas p_coordinador ON p_coordinador .nId_Persona = c.nId_Persona

JOIN Clientes c2 ON c.nId_Cliente = c2.nId_Cliente

JOIN Personas p2 ON c2.nId_Persona = p2.nId_Persona

JOIN Configs co ON co.sCodigo = CONVERT(VARCHAR,r.nEstado) AND co.sTabla = 'ESTADO_REQUERIMIENTO'

LEFT JOIN Tareas t2 ON t2.nId_Proyecto = p.nId_Proyecto AND t2.nId_Requerimiento = r.nId_Requerimiento

GROUP BY

r.nId_Requerimiento,

r.sNombre ,

p.nId_Proyecto,

r.nId_Lider ,

p.nId_Lider,

pLider.sPrimer_Nombre,

pLider.sApe_Paterno,

pDirector.sPrimer_Nombre,

pDirector.sApe_Paterno,

p.sNombre,

r.dFecha_Inicio,

r.dFecha_Fin,

r.dFecha_Modificacion_Estado,

p_coordinador.sPersona_Nombre,

p2.sPersona_Nombre,

r.nEstado,

co.sDescripcion ,

r.dDatetime_Creador,

r.nUsuario_Creador,

r.sCodigo,

p.sId_Tipo_Servicio;
```