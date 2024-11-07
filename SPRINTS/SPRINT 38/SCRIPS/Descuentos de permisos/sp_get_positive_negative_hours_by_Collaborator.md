```sql
create or alter PROCEDURE sp_get_positive_negative_hours_by_Collaborator(
    @dFecha DATETIME,
    @sIds_Colaborador NVARCHAR(MAX) = NULL -- Assuming you pass a comma-separated string
)AS
BEGIN
    DECLARE @nId_Collaborators TABLE (nId INT)
	
	------
	declare @dFecha_cierre varchar(20)
	select top 1 @dFecha_cierre =  dFecha_cierre from closure_request order by nid_cierre desc
	------

    -- Split the collaborator IDs into a table
    INSERT INTO @nId_Collaborators
    SELECT value
    FROM STRING_SPLIT(@sIds_Colaborador, ',');

    DECLARE @tTardanzas TABLE (nId_Colaborador INT, dTardanza_Con_Tolerancia INT, dTardanza_Sin_Tolerancia INT)
    DECLARE @tAfavorContra TABLE (nId_Colaborador INT, nPermisos INT, nAFavor INT)
    DECLARE @tExtra TABLE (nId_Colaborador INT, nExtra INT)

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
      AND YEAR(a.dFecha_Asistencia) = YEAR(@dFecha)
      AND tsm.nId_Colaborador IN (SELECT nId FROM @nId_Collaborators)
    GROUP BY tsm.nId_Colaborador

    -- Insert afavor and contra data into the table
    INSERT INTO @tAfavorContra
    SELECT
        rht.nId_Colaborador,
        rht.CantidadTotalSuma AS nPermisos,
        rht.CantidadTotalResta AS nAFavor
    FROM dbo.fn_get_request_hours_by_Transaction(@sIds_Colaborador, 0, 0, NULL, @dFecha) rht

    -- Insert extra task data into the table
    INSERT INTO @tExtra
    SELECT
        nId_Colaborador,
        nMinutos
    FROM Tareas
    WHERE sTipo_Hora = 2
      AND nEstado_Tarea = 3
      AND nEstado_Pago IS NULL
      AND nId_Colaborador IN (SELECT nId FROM @nId_Collaborators)
      AND YEAR(dFecha_Registro) = YEAR(@dFecha)

    -- Select results by joining the temporary tables
    SELECT DISTINCT
        ISNULL(p.nId_Colaborador, ISNULL(t.nId_Colaborador, e.nId_Colaborador)) AS nId_Colaborador,
        ISNULL(t.dTardanza_Con_Tolerancia, 0) AS dTardanza_Con_Tolerancia,
        ISNULL(t.dTardanza_Sin_Tolerancia, 0) AS dTardanza_Sin_Tolerancia,
        ISNULL(p.nPermisos, 0) AS nPermisos,
        ISNULL(p.nAFavor, 0) AS nAFavor,
        ISNULL(p.nPermisos, 0) + ISNULL(t.dTardanza_Con_Tolerancia, 0) AS nEnContra,
        ISNULL(e.nExtra, 0) AS nExtra,
		------
		isnull(@dFecha_cierre, '') dFecha_cierre
		------
    FROM @tAfavorContra p
    LEFT JOIN @tTardanzas t ON t.nId_Colaborador = p.nId_Colaborador
    LEFT JOIN @tExtra e ON e.nId_Colaborador = p.nId_Colaborador
END
go
```