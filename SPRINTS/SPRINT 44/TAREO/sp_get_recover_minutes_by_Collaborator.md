```sql
CREATE PROCEDURE sp_get_recover_minutes_by_Collaborator(

@dFecha DATETIME,

@sIds_Colaborador NVARCHAR(MAX) = NULL -- Assuming you pass a comma-separated string

)

AS

--DECLARE @dFecha DATETIME ='2025-04-13'

--DECLARE @sIds_Colaborador NVARCHAR(MAX) = '6'

  

BEGIN

  

  

DECLARE @nId_Collaborators TABLE (nId INT)

-- Tabla temporal para almacenar el resultado

IF OBJECT_ID('tempdb..#Total_Minutos_Recuperar') IS NOT NULL

DROP TABLE #Total_Minutos_Recuperar;

  

CREATE TABLE #Total_Minutos_Recuperar (

dTotal_Minutos_Recuperar INT

)

  

------

declare @dFecha_cierre varchar(20)

select top 1 @dFecha_cierre = dFecha_cierre from closure_request order by nid_cierre desc

------

  

-- Split the collaborator IDs into a table

INSERT INTO @nId_Collaborators

SELECT value

FROM STRING_SPLIT(@sIds_Colaborador, ',');

  

DECLARE @tTardanzas TABLE (nId_Colaborador INT, dTardanza_Con_Tolerancia INT, dTardanza_Sin_Tolerancia INT)

DECLARE @tAfavorContra TABLE (nId_Colaborador INT, nPermisos INT, nAFavor INT)

DECLARE @tExtra TABLE (nId_Colaborador INT, nExtra INT)

DECLARE @tCompensacion TABLE (nId_Colaborador INT, nCompensacion INT)

DECLARE @tBolsa TABLE (nId_Colaborador INT, nBolsa INT)

  

  

-- Insert tardiness data into the table

INSERT INTO @tTardanzas

SELECT

tsm.nId_Colaborador,

SUM(CASE WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad = 3

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nTipo_Transaccion = 2

AND tsm.nId_Sub_Tipo_Entidad = 3

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos ELSE 0 END) AS dTardanza_Con_Tolerancia,

SUM(CASE WHEN tsm.nTipo_Transaccion = 1

AND tsm.nId_Sub_Tipo_Entidad = 4

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos ELSE 0 END) -

SUM(CASE WHEN tsm.nTipo_Transaccion = 2

AND tsm.nId_Sub_Tipo_Entidad = 4

AND tsm.nId_Cierre IS NULL

THEN tsm.dCantidad_Minutos ELSE 0 END) AS dTardanza_Sin_Tolerancia

FROM Transacciones_Saldo_Mins tsm

LEFT JOIN Asistencias a ON a.nId_Asistencia = tsm.nId_Entidad

WHERE tsm.nEstado_Transaccion = 1

AND tsm.nId_Tipo_Entidad = 2

AND a.dFecha_Asistencia <= CONVERT(DATE,@dFecha)

AND tsm.nId_Colaborador IN (SELECT nId FROM @nId_Collaborators)

GROUP BY tsm.nId_Colaborador

  

-- Insert afavor and contra data into the table

INSERT INTO @tAfavorContra

SELECT

rht.nId_Colaborador,

rht.CantidadTotalSuma AS nPermisos,

rht.CantidadTotalResta AS nAFavor

  

FROM dbo.fn_get_request_hours_by_Transaction(@sIds_Colaborador, 0, 0, 0, NULL, CONVERT(DATE,@dFecha)) rht

  

-- Insert extra task data into the table

INSERT INTO @tExtra

SELECT

nId_Colaborador,

SUM(nMinutos)

FROM Tareas

WHERE sTipo_Hora = 2

AND nEstado_Tarea = 3

AND nEstado_Pago IS NULL

AND nId_Colaborador IN (

CASE

WHEN @sIds_Colaborador IS NULL THEN nId_Colaborador

ELSE (SELECT nId FROM @nId_Collaborators)

END

)

AND dFecha_Registro <= CONVERT(DATE,@dFecha)

group by nId_Colaborador

  

  

INSERT INTO @tCompensacion

SELECT

tsm.nId_Colaborador,

SUM(tsm.dCantidad_Minutos) AS nCompensacion

FROM transacciones_saldo_mins tsm

WHERE nId_Tipo_Entidad = 9

AND nId_Sub_Tipo_Entidad = 8

AND nId_Cierre IS NULL

AND tsm.nId_Colaborador IN (SELECT nId FROM @nId_Collaborators)

AND CONVERT(DATE, tsm.dDatetime_Creacion) <= CONVERT(DATE, @dFecha)

GROUP BY tsm.nId_Colaborador

  

INSERT INTO @tBolsa

SELECT

tsm.nId_Colaborador,

tsm.dCantidad_Minutos

FROM transacciones_saldo_mins tsm

WHERE nId_Tipo_Entidad = 5

AND nId_Sub_Tipo_Entidad = 8

AND nId_Cierre IS NULL

AND tsm.nId_Colaborador IN (SELECT nId FROM @nId_Collaborators)

AND CONVERT(DATE, tsm.dDatetime_Creacion) <= CONVERT(DATE, @dFecha)

  

INSERT INTO #Total_Minutos_Recuperar(dTotal_Minutos_Recuperar)

SELECT

(ISNULL(ISNULL(t.dTardanza_Con_Tolerancia, 0) + ISNULL(p.nPermisos, 0) - ISNULL(p.nAFavor, 0), 0))

FROM @tAfavorContra p

FULL OUTER JOIN @tTardanzas t ON t.nId_Colaborador = COALESCE(p.nId_Colaborador, t.nId_Colaborador)

--Retornar valor---

SELECT * FROM #Total_Minutos_Recuperar

END;
```
