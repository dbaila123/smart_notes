```SQL
SELECT

r.nId_Requerimiento,

r.sNombre as sNombre_Requerimiento,

r.sPrefijo,

r.nId_Proyecto,

r.nId_Lider as nId_Lider_Rq,

p.nId_Lider AS nLiderPrincipal,

CONCAT(pLider.sPrimer_Nombre, ' ', pLider.sApe_Paterno) as sNombreLider,

CONCAT(pDir.sPrimer_Nombre, ' ', pDir.sApe_Paterno) as sNombreDirector,

p.sNombre AS sNombre_Proyecto,

CONCAT(p.sCodigo, ' ', p.sNombre) AS sNombre_Proyecto_ConPrefijo,

CONVERT(DATE, r.dFecha_Inicio) AS dFecha_Inicio,

dbo.fn_get_Real_End_Date_Requirement(r.nId_Requerimiento) dFechaFinRqReal,

r.dFecha_Modificacion_Estado,

p_coordinador.sPersona_Nombre AS Nombre_Coordinador,

p2.sPersona_Nombre AS Nombre_Cliente,

r.nEstado,

prmc.nEstimado as nEstimacion,

co.sDescripcion AS sEstado_Requerimiento,

CASE WHEN

(select count(*) from justificaciones_Proyectos_Requerimientos jr where jr.nId_Entidad = r.nId_Requerimiento and jr.sEntidad = 'REQUERIMIENTO') > 0

THEN convert(bit,1)

ELSE convert(bit,0)

END as bJustificacion,

r.dDatetime_Creador AS dFecha_Creacion,

CONCAT(pCrea.sPrimer_Nombre, ' ', pCrea.sApe_Paterno) AS sNombre_Usuario_Creador,

cSer.sDescripcion AS sTipo_Servcio_Req,

r.sCodigo

FROM Requerimientos r

JOIN Proyectos p ON r.nId_Proyecto = p.nId_Proyecto

JOIN Colaboradores cLider ON r.nId_Lider = cLider.nId_Colaborador

JOIN Personas pLider ON pLider.nId_Persona = cLider.nId_Persona

JOIN Colaboradores cDir ON p.nId_Director = cDir.nId_Colaborador

JOIN Personas pDir ON pDir.nId_Persona = cDir.nId_Persona

JOIN Coordinadores_Proyectos cp ON cp.nId_Proyecto = p.nId_Proyecto

JOIN Coordinadores c ON c.nId_Coordinador = cp.nId_Coordinador

JOIN Personas p_coordinador ON p_coordinador.nId_Persona = c.nId_Persona

JOIN Clientes c2 ON c.nId_Cliente = c2.nId_Cliente

JOIN Personas p2 ON c2.nId_Persona = p2.nId_Persona

JOIN pryRqMinutosCalculados prmc ON prmc.nId_Requerimiento = r.nId_Requerimiento

JOIN Configs co ON co.sCodigo = CONVERT(VARCHAR,r.nEstado) AND co.sTabla = 'ESTADO_REQUERIMIENTO'

JOIN Usuarios uCrea ON uCrea.nId_Usuario = r.nUsuario_Creador

JOIN Personas pCrea ON pCrea.nId_Persona = uCrea.nId_Usuario

JOIN Configs cSer ON cSer.sCodigo = CONVERT(VARCHAR, p.sId_Tipo_Servicio) AND cSer.sTabla = 'TIPO_SERVICIO'

  

  

SELECT * from Requerimientos r

  

CREATE FUNCTION fn_get_Real_End_Date_Requirement(

@nId_Rq INT

)

RETURNS DATE

AS

BEGIN

DECLARE @dFechaFin DATE;

  

SELECT @dFechaFin = MAX(t.dFecha_Registro)

FROM Tareas t

WHERE t.nId_Requerimiento = @nId_Rq;

  

RETURN @dFechaFin;

END;

  

  

  

UPDATE p

SET sCodigo = p.nId_Proyecto

From Proyectos p

WHERE P.sCodigo is NULL

  

  

  

UPDATE p

SET sNombre = p.sNombre_Proyecto_Sin_Prefijo

From Proyectos p

--WHERE P.sCodigo is NULL
```