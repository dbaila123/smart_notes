```sql
-- dbo.v_Listado_Proyectos_con_requerimientos source
CREATE VIEW v_Listado_Proyectos_con_requerimientos

AS

SELECT DISTINCT p.nId_Proyecto,

pl.nPrincipal,

pl.nId_Lider as nLider /*Se muestra el lider principal o el lider segundario para poder obteber el proyecto*/ ,

p.sNombre_Proyecto_Sin_Prefijo,

p.sNombre_Proyecto_Sin_Prefijo as sNombre_Proyecto_Detalle,

CONCAT(

(SELECT c.sDescripcion from Configs c

where c.sTabla = 'PREFIJO_PROYECTO'

and c.sCodigo = CAST(p.sPrefijo as VARCHAR(10))),

' ',

p.sCodigo,

' ',

p.sNombre_Proyecto_Sin_Prefijo

) as sNombre_Proyecto_Lista,

  

p.sDescripcion,

p.nId_Lider as nLiderPrincipal,

p.nTipo_Acceso,

p.sCodigo as sCodigo_Proyecto,

p.sPrefijo AS sPrefijo_Codigo,

(SELECT c.sDescripcion from Configs c where c.sTabla = 'PREFIJO_PROYECTO' and c.sCodigo = CAST(p.sPrefijo as VARCHAR(10))) as sPrefijo,

(SELECT c.sDescripcion from Configs c where sTabla = 'TIPO_ACCESO_PROYECTO' and c.sCodigo = CAST(p.nTipo_Acceso as VARCHAR(10))) as sTipo_Acceso,

concat(pLider.sPrimer_Nombre, ' ', pLider.sApe_Paterno) sLider /*Se muestra el Lider Principal*/,

p.nId_Director as nDirector,

concat(pDirector.sPrimer_Nombre, ' ', pDirector.sApe_Paterno) sDirector,

cp.nId_Coordinador as nCoordinador,

pCoodinador.sPersona_Nombre as sCoordinador,

(SELECT c.sCodigo from Configs c where sTabla = 'TIPO_SERVICIO' and c.sCodigo = p.sId_Tipo_Servicio ) as nTipo_Servicio,

(SELECT c.sDescripcion from Configs c where sTabla = 'TIPO_SERVICIO' and c.sCodigo = p.sId_Tipo_Servicio ) as sTipo_Servicio,

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

(SELECT c.sDescripcion from Configs c where sTabla = 'ESTADO_PROYECTO' and c.sCodigo = p.sEstado_Proyecto ) as sEstado_Descripcion,

pCliente.sPersona_Nombre sCliente,

CASE

WHEN (select count(*)

from justificaciones_Proyectos_Requerimientos jr

where jr.nId_Entidad = p.nId_Proyecto and jr.sEntidad = 'PROYECTO') > 0

THEN convert(bit,1)

ELSE convert(bit,0)

END bJustificacion,

(SELECT ISNULL (SUM(rc.nEstimacion),0) from Requerimientos r

JOIN Requerimientos_Categoria rc on rc.nId_Requerimiento = r.nId_Requerimiento

where r.nId_Proyecto = p.nId_Proyecto) nEstimacion

----isnull(SUM(rc.nEstimacion),0) AS nEstimacion

FROM Proyectos p

join Proyecto_Lider pl on p.nId_Proyecto = pl.nId_Proyecto and pl.nEstado = 1--PRY_LIDER

join Colaboradores cLider on cLider.nId_Colaborador = p.nId_Lider --LIDER

join Personas pLider on pLider.nId_Persona = cLider.nId_Persona ---LIDER

join Colaboradores cDirector on cDirector.nId_Colaborador = p.nId_Director --DIRECTOR

join Personas pDirector on pDirector.nId_Persona = cDirector.nId_Persona ---DIRECTOR

left join Coordinadores_Proyectos cp on cp.nId_Proyecto = p.nId_Proyecto --COODINADOR_PROYECTO

left join Coordinadores coodi on coodi.nId_Coordinador = cp.nId_Coordinador --COODINADOR

left join Personas pCoodinador on pCoodinador.nId_Persona = coodi.nId_Persona --COODINAROR

join Clientes cliente on cliente.nId_Cliente = coodi.nId_Cliente --CLIENTE

join Personas pCliente on pCliente.nId_Persona = cliente.nId_Persona --CLIENTE

left join Tareas t on t.nId_Proyecto = p.nId_Proyecto

GROUP by p.nId_Proyecto,

p.sPrefijo,

p.sCodigo,

pl.nPrincipal,

p.nId_Lider,

p.sNombre ,

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

P.dFecha_Fin,

P.dFecha_Modificacion_Estado,

P.dDatetime_Creador,

p.nUsuario_Creador,

p.sEstado_Proyecto,

p.nEstado,

p.nTipo_Acceso,

pCliente.sPersona_Nombre;
```