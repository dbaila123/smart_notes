```SQL
CREATE OR ALTER VIEW v_Listado_Proyectos_con_requerimientos

AS

SELECT DISTINCT

p.nId_Proyecto,

pl.nPrincipal,

pl.nId_Lider as nLider /*Se muestra el lider principal o el lider segundario para poder obteber el proyecto*/ ,

p.sNombre_Proyecto_Sin_Prefijo,

CONCAT(cPrefProy.sDescripcion, ' - ', p.sNombre_Proyecto_Sin_Prefijo) as sNombre_Proyecto_Detalle,

/*CASE

WHEN p.sCodigo!= '-' THEN CONCAT(p.sCodigo, ' ', p.sNombre_Proyecto_Sin_Prefijo)

ELSE CONCAT((SELECT c.sDescripcion from Configs c where c.sTabla = 'PREFIJO_PROYECTO' and c.sCodigo = CAST(p.sPrefijo as VARCHAR(10))), ' - ', p.sNombre_Proyecto_Sin_Prefijo)

END as sNombre_Proyecto_Lista,*/

p.sNombre as sNombre_Proyecto_Lista,

p.sDescripcion,

p.nId_Lider as nLiderPrincipal,

p.nTipo_Acceso,

p.sCodigo as sCodigo_Proyecto,

p.sPrefijo AS sPrefijo_Codigo,

cPrefProy.sDescripcion as sPrefijo,

cTipoAcceso.sDescripcion as sTipo_Acceso,

concat(pLider.sPrimer_Nombre, ' ', pLider.sApe_Paterno) sLider /*Se muestra el Lider Principal*/,

p.nId_Director as nDirector,

concat(pDirector.sPrimer_Nombre, ' ', pDirector.sApe_Paterno) sDirector,

cp.nId_Coordinador as nCoordinador,

pCoodinador.sPersona_Nombre as sCoordinador,

cTipoServ.sCodigo as nTipo_Servicio,

cTipoServ.sDescripcion as sTipo_Servicio,

convert(datetime2(7), p.dFecha_Inicio) AS dFecha_Inicio,

MIN(t.dFecha_Registro) dFechaInicioPryReal,

--(SELECT MIN(dFecha_Registro) from Tareas t where nId_Proyecto = p.nId_Proyecto) AS dFechaInicioPryReal,

CASE

WHEN p.dFecha_Fin < (select MAX(dFecha_Fin) from Requerimientos r where r.nId_Proyecto = p.nId_Proyecto)

THEN (select MAX(dFecha_Fin) from Requerimientos r where r.nId_Proyecto = p.nId_Proyecto)

WHEN p.dFecha_Fin is NULL

THEN (select MAX(dFecha_Fin) from Requerimientos r where r.nId_Proyecto = p.nId_Proyecto)

ELSE p.dFecha_Fin

END dFecha_Fin,

max(t.dFecha_Registro) dFechaFinPryReal,

--(SELECT max(dFecha_Registro) from Tareas t where nId_Proyecto = p.nId_Proyecto) AS dFechaFinPryReal,

p.dFecha_Modificacion_Estado,

convert(datetime2(7), p.dDatetime_Creador) AS dFecha_Creacion,

CASE

WHEN p.nUsuario_Creador = 172 or p.nUsuario_Creador = 666

THEN 'SMART'

ELSE

(

SELECT TOP 1

CONCAT(tb_person_creator.sPrimer_Nombre, ' ', tb_person_creator.sApe_Paterno)

FROM Usuarios tb_user_creator

JOIN Personas tb_person_creator

ON tb_user_creator.nId_Persona = tb_person_creator.nId_Persona

WHERE tb_user_creator.nId_Usuario = p.nUsuario_Creador

)

END AS sNombre_Usuario_Creador,

p.sEstado_Proyecto AS nEstado,

cEstadoProy.sDescripcion as sEstado_Descripcion,

pCliente.sPersona_Nombre sCliente,

CASE

WHEN (select count(*)

from justificaciones_Proyectos_Requerimientos jr

where jr.nId_Entidad = p.nId_Proyecto and jr.sEntidad = 'PROYECTO') > 0

THEN convert(bit,1)

ELSE convert(bit,0)

END bJustificacion,

(SELECT ISNULL(SUM(rc.nEstimacion),0)

from Requerimientos r

JOIN Requerimientos_Categoria rc on rc.nId_Requerimiento = r.nId_Requerimiento

where r.nId_Proyecto = p.nId_Proyecto) nEstimacion

----isnull(SUM(rc.nEstimacion),0) AS nEstimacion

FROM Proyectos p

JOIN Proyecto_Lider pl ON p.nId_Proyecto = pl.nId_Proyecto and pl.nEstado = 1--PRY_LIDER

JOIN Colaboradores cLider ON cLider.nId_Colaborador = p.nId_Lider --LIDER

JOIN Personas pLider ON pLider.nId_Persona = cLider.nId_Persona ---LIDER

JOIN Colaboradores cDirector ON cDirector.nId_Colaborador = p.nId_Director --DIRECTOR

JOIN Personas pDirector ON pDirector.nId_Persona = cDirector.nId_Persona ---DIRECTOR

LEFT JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto --COODINADOR_PROYECTO

LEFT JOIN Coordinadores coodi ON coodi.nId_Coordinador = cp.nId_Coordinador --COODINADOR

LEFT JOIN Personas pCoodinador ON pCoodinador.nId_Persona = coodi.nId_Persona --COODINAROR

JOIN Clientes cliente ON cliente.nId_Cliente = coodi.nId_Cliente --CLIENTE

JOIN Personas pCliente ON pCliente.nId_Persona = cliente.nId_Persona --CLIENTE

LEFT JOIN Tareas t ON t.nId_Proyecto = p.nId_Proyecto

JOIN Configs cPrefProy ON cPrefProy.sTabla = 'PREFIJO_PROYECTO'

AND cPrefProy.sCodigo = CAST(p.sPrefijo as VARCHAR(10))

JOIN Configs cTipoAcceso ON cTipoAcceso.sTabla = 'TIPO_ACCESO_PROYECTO'

AND cTipoAcceso.sCodigo = CAST(p.nTipo_Acceso as VARCHAR(10))

JOIN Configs cTipoServ ON cTipoServ.sTabla = 'TIPO_SERVICIO'

AND cTipoServ.sCodigo = p.sId_Tipo_Servicio

JOIN Configs cEstadoProy ON cEstadoProy.sTabla = 'ESTADO_PROYECTO'

AND cEstadoProy.sCodigo = p.sEstado_Proyecto

GROUP BY

p.nId_Proyecto,

p.sPrefijo,

p.sCodigo,

pl.nPrincipal,

p.nId_Lider,

p.sNombre,

p.sNombre_Proyecto_Sin_Prefijo,

p.sDescripcion,

pl.nId_Lider,

p.nId_Director,

p.sId_Tipo_Servicio,

pLider.sPrimer_Nombre,

pLider.sApe_Paterno,

pDirector.sPrimer_Nombre,

pDirector.sApe_Paterno,

cp.nId_Coordinador,

pCoodinador.sPersona_Nombre,

p.dFecha_Inicio,

p.dFecha_Fin,

p.dFecha_Modificacion_Estado,

p.dDatetime_Creador,

p.nUsuario_Creador,

p.sEstado_Proyecto,

p.nEstado,

p.nTipo_Acceso,

pCliente.sPersona_Nombre,

cPrefProy.sDescripcion,

cTipoAcceso.sDescripcion,

cTipoServ.sCodigo,

cTipoServ.sDescripcion,

cEstadoProy.sDescripcion;