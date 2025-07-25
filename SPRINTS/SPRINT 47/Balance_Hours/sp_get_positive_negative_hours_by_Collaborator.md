```sql
CREATE PROCEDURE [dbo].[sp_get_positive_negative_hours_by_Collaborator](

@dFecha DATETIME,

@sIds_Colaborador NVARCHAR(MAX) = NULL -- Assuming you pass a comma-separated string

  

) AS

BEGIN

  

DECLARE @nId_Collaborators TABLE (nId INT)

DECLARE @dFecha_cierre varchar(20)

  

------

SELECT TOP 1 @dFecha_cierre = dFecha_cierre from closure_request order by nid_cierre desc

------

  

-- Split the collaborator IDs into a table

INSERT INTO @nId_Collaborators

SELECT value

FROM STRING_SPLIT(@sIds_Colaborador, ',');

  

DECLARE @tTardanzas TABLE (nId_Colaborador INT, dTardanza_Con_Tolerancia INT, dTardanza_Sin_Tolerancia INT)

-- Nuevas tablas para separar las horas extras por estado

DECLARE @tExtra TABLE (nId_Colaborador INT, nExtraApro INT, nExtraPend INT)

DECLARE @tBolsa TABLE (nId_Colaborador INT, nBolsa INT)

  

-- Insertar datos de tardanzas en la tabla temporal @tTardanzas RECUPERABLES Y NO RECUPERABLES

INSERT INTO @tTardanzas

SELECT

tsm.nId_Colaborador,

SUM(CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad = 3

AND tsm.nId_Cierre IS NULL

  

THEN tsm.dCantidad_Minutos

ELSE 0

END) -

SUM(CASE

WHEN tsm.nTipo_Transaccion = 2

AND tsm.nId_Sub_Tipo_Entidad = 3

AND tsm.nId_Cierre IS NULL

  

THEN tsm.dCantidad_Minutos

ELSE 0

END) AS dTardanza_Con_Tolerancia,

SUM(CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad = 4

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) -

SUM(CASE

WHEN tsm.nTipo_Transaccion = 2

AND tsm.nId_Sub_Tipo_Entidad = 4

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) AS dTardanza_Sin_Tolerancia

FROM Transacciones_Saldo_Mins tsm

LEFT JOIN Asistencias a ON a.nId_Asistencia = tsm.nId_Entidad

WHERE tsm.nEstado_Transaccion = 1

AND tsm.nId_Tipo_Entidad = 2

AND a.dFecha_Asistencia <= CONVERT(DATE, @dFecha)

AND (

NOT EXISTS (SELECT 1 FROM @nId_Collaborators)

OR tsm.nId_Colaborador IN (SELECT nId FROM @nId_Collaborators)

)

  

GROUP BY tsm.nId_Colaborador;

  

--Permisos para pre planilla

DECLARE @dFecha_Cierre_Planificado DATE;

  

SELECT TOP 1

@dFecha_Cierre_Planificado = car.dFecha_Cierre_Planificado

FROM cierreAsistenciaReportes car

ORDER BY car.nId_Cierre desc

  

SELECT

t.nId_Colaborador,

STRING_AGG(t.nId_Solicitud, '-') AS sSolicitudes,

SUM(nPermisos) AS sTiempo

INTO #PrePlanilla_Permiso

FROM (

SELECT

tsm.nId_Colaborador,

tsm.nId_Entidad AS nId_Solicitud,

SUM(CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad IN (6,7)

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) AS nPermisos

FROM Transacciones_Saldo_Mins tsm

JOIN Solicitudes st ON tsm.nId_Entidad = st.nId_Solictud

WHERE tsm.nEstado_Transaccion = 1

AND tsm.nId_Tipo_Entidad = 1

AND st.nEstado_Solicitud = 3

AND CONVERT(DATE, tsm.dDatetime_Creacion) <= @dFecha

AND DATEADD(DAY, dbo.fn_get_closing_days_by_request_date(CAST(tsm.dDatetime_Creacion AS DATE)), tsm.dDatetime_Creacion) <= @dFecha_Cierre_Planificado

GROUP BY tsm.nId_Colaborador, tsm.nId_Entidad, tsm.dDatetime_Creacion, st.nEstado_Solicitud

HAVING SUM(

CASE

WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad IN (6,7)

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos

ELSE 0

END) > 0

) t GROUP BY t.nId_Colaborador

  

  

-------- Insert extra hours data into tables (aprobadas - state 3)

INSERT INTO @tExtra

SELECT

nId_Colaborador,

SUM(CASE WHEN nEstado_Tarea = 3 THEN nMinutos ELSE 0 END) AS nExtraApro,

SUM(CASE WHEN nEstado_Tarea IN (1, 9) THEN nMinutos ELSE 0 END) AS nExtraPend

FROM Tareas

WHERE sTipo_Hora = 2

AND nEstado_Pago IS NULL

AND (

NOT EXISTS (SELECT 1 FROM @nId_Collaborators)

OR nId_Colaborador IN (SELECT nId FROM @nId_Collaborators)

)

AND dFecha_Registro <= CONVERT(DATE, @dFecha)

GROUP BY nId_Colaborador;

  

INSERT INTO @tBolsa

SELECT

tsm.nId_Colaborador,

tsm.dCantidad_Minutos

FROM transacciones_saldo_mins tsm

WHERE nId_Tipo_Entidad = 5

AND nId_Sub_Tipo_Entidad = 8

AND nId_Cierre IS NULL

AND (

NOT EXISTS (SELECT 1 FROM @nId_Collaborators)

  

OR tsm.nId_Colaborador IN (SELECT nId FROM @nId_Collaborators)

)

AND CONVERT(DATE, tsm.dDatetime_Creacion) <= CONVERT(DATE, @dFecha)

  

MERGE INTO Total_Min_Apro_Pend AS target

USING (

SELECT

col.nId_Colaborador AS nId_Colaborador,

per.sPersona_Nombre AS sNombre_Colaborador,

ISNULL(t.dTardanza_Con_Tolerancia, 0) AS dTardanza_Con_Tolerancia,

ISNULL(t.dTardanza_Sin_Tolerancia, 0) AS nTotal_Min_No_Recuperables,

ISNULL(ta.nPermisosApro, 0) AS nTotal_Min_Contra_Per_Apro,

ISNULL(ta.nTotal_Favor_CompeRecu_A, 0) AS nTotal_Min_Favor_CompRecu_Apro,

ISNULL(ta.nPermisosPen, 0) AS nTotal_Min_Contra_Per_Pend,

ISNULL(ta.nAFavorPen, 0) AS nTotal_Min_Favor_CompRecu_Pend,

ISNULL(ea.nExtraApro, 0) AS nTotal_Min_Favor_Extra_Apro,

ISNULL(ea.nExtraPend, 0) AS nTotal_Min_Favor_Extra_Pend,

  

ISNULL(ea.nExtraApro, 0) + ISNULL(ea.nExtraPend, 0) AS nTotal_Min_Favor_Extra_AP,

ISNULL(ta.nAFavorAP, 0) AS nTotal_Min_Favor_CompRecu_AP,

ISNULL(ta.nPermisosAP, 0) AS nTotal_Min_Contra_Per_AP,

ISNULL(bs.nBolsa, 0) AS nTotal_Bolsa_Cierre,

ISNULL(ta.nPermisosApro, 0) + ISNULL(t.dTardanza_Con_Tolerancia, 0) + ISNULL(t.dTardanza_Sin_Tolerancia, 0) AS nTotal_Contra,

ISNULL(ta.nAFavorAP, 0) + ISNULL(ea.nExtraApro, 0) + ISNULL(ea.nExtraPend, 0) + ISNULL(bs.nBolsa, 0) AS nTotal_Favor,

ISNULL(t.dTardanza_Con_Tolerancia,0)+ ISNULL(ta.nPermisosApro, 0)- ISNULL(ta.nAFavorAP,0) - isnull(bs.nBolsa,0) AS nTotal_Minutos_Recuperar,

ISNULL(pp.sTiempo, 0) AS nPrePlanilla_Tiempo_Permisos

FROM Colaboradores col

JOIN Personas per ON per.nId_Persona = col.nId_Persona

FULL OUTER JOIN dbo.fn_get_request_hours_by_Transaction_All(@sIds_Colaborador, 0, 0, 0, NULL, CONVERT(DATE, GETDATE())) ta ON col.nId_Colaborador = ta.nId_Colaborador

FULL OUTER JOIN @tExtra ea ON ea.nId_Colaborador = col.nId_Colaborador

FULL OUTER JOIN @tTardanzas t ON t.nId_Colaborador = col.nId_Colaborador

FULL OUTER JOIN @tBolsa bs ON bs.nId_Colaborador = col.nId_Colaborador

FULL OUTER JOIN #PrePlanilla_Permiso pp on col.nId_Colaborador = pp.nId_Colaborador

--WHERE col.nEstado_Colaborador = 1

AND (

NOT EXISTS (SELECT 1 FROM @nId_Collaborators)

  

OR col.nId_Colaborador IN (SELECT nId FROM @nId_Collaborators)

)

) AS source

ON target.nId_Colaborador = source.nId_Colaborador

  

WHEN MATCHED THEN

UPDATE SET

target.sNombre_Colaborador = source.sNombre_Colaborador,

target.dTardanza_Con_Tolerancia = source.dTardanza_Con_Tolerancia,

target.nTotal_Min_No_Recuperables = source.nTotal_Min_No_Recuperables,

target.nTotal_Min_Contra_Per_Apro = source.nTotal_Min_Contra_Per_Apro,

target.nTotal_Min_Favor_CompRecu_Apro = source.nTotal_Min_Favor_CompRecu_Apro,

target.nTotal_Min_Contra_Per_Pend = source.nTotal_Min_Contra_Per_Pend,

target.nTotal_Min_Favor_CompRecu_Pend = source.nTotal_Min_Favor_CompRecu_Pend,

target.nTotal_Min_Favor_Extra_Apro = source.nTotal_Min_Favor_Extra_Apro,

target.nTotal_Min_Favor_Extra_Pend = source.nTotal_Min_Favor_Extra_Pend,

target.nTotal_Min_Favor_Extra_AP = source.nTotal_Min_Favor_Extra_AP,

target.nTotal_Min_Favor_CompRecu_AP = source.nTotal_Min_Favor_CompRecu_AP,

target.nTotal_Min_Contra_Per_AP = source.nTotal_Min_Contra_Per_AP,

target.nTotal_Bolsa_Cierre = source.nTotal_Bolsa_Cierre,

target.nTotal_Contra = source.nTotal_Contra,

target.nTotal_Favor = source.nTotal_Favor,

target.nTotal_Minutos_Recuperar = source.nTotal_Minutos_Recuperar,

target.nPrePlanilla_Tiempo_Permisos = source.nPrePlanilla_Tiempo_Permisos

  

WHEN NOT MATCHED BY TARGET THEN

INSERT (

nId_Colaborador,

sNombre_Colaborador,

dTardanza_Con_Tolerancia,

nTotal_Min_No_Recuperables,

nTotal_Min_Contra_Per_Apro,

nTotal_Min_Favor_CompRecu_Apro,

nTotal_Min_Contra_Per_Pend,

nTotal_Min_Favor_CompRecu_Pend,

nTotal_Min_Favor_Extra_Apro,

nTotal_Min_Favor_Extra_Pend,

nTotal_Min_Favor_Extra_AP,

nTotal_Min_Favor_CompRecu_AP,

nTotal_Min_Contra_Per_AP,

nTotal_Bolsa_Cierre,

nTotal_Contra,

nTotal_Favor,

nTotal_Minutos_Recuperar,

nPrePlanilla_Tiempo_Permisos

) VALUES (

source.nId_Colaborador,

source.sNombre_Colaborador,

source.dTardanza_Con_Tolerancia,

source.nTotal_Min_No_Recuperables,

source.nTotal_Min_Contra_Per_Apro,

source.nTotal_Min_Favor_CompRecu_Apro,

source.nTotal_Min_Contra_Per_Pend,

source.nTotal_Min_Favor_CompRecu_Pend,

source.nTotal_Min_Favor_Extra_Apro,

source.nTotal_Min_Favor_Extra_Pend,

source.nTotal_Min_Favor_Extra_AP,

source.nTotal_Min_Favor_CompRecu_AP,

source.nTotal_Min_Contra_Per_AP,

source.nTotal_Bolsa_Cierre,

source.nTotal_Contra,

source.nTotal_Favor,

source.nTotal_Minutos_Recuperar,

source.nPrePlanilla_Tiempo_Permisos

);

  

END
```