```SQL
CREATE PROCEDURE [dbo].[sp_create_closing_attendance] @nId_User INT,
 @dFecha_Inicio_Asistencia DATE,
 @dFecha_Fin_Asistencia DATE,
 @dFecha_Inicio_Tardanza DATE,
 @dFecha_Fin_Tardanza DATE,
 @dFecha_Periodo DATE,
 @dFecha_Cierre_Planificado DATE
AS
BEGIN TRY
 BEGIN TRANSACTION
 DECLARE @Hoy DATE = GETDATE();
 DECLARE @nId_Cierre INT;

 --exec sp_update_attendance_massive_or_individual null, @dFecha_Inicio_Asistencia, @dFecha_Fin_Asistencia, 172

 EXEC sp_closing_attendance_report_insert
 @nId_Cierre = @nId_Cierre OUTPUT,
 @dFecha_Inicio_Asistencia = @dFecha_Inicio_Asistencia,
 @dFecha_Fin_Asistencia = @dFecha_Fin_Asistencia,
 @dFecha_Inicio_Tardanza = @dFecha_Inicio_Tardanza,
 @dFecha_Fin_Tardanza = @dFecha_Fin_Tardanza,
 @dFecha_Periodo = @dFecha_Periodo,
 @nUsuario_Creador = @nId_User,
 @dDatetime_Creador = @Hoy,
 @dFecha_Cierre_Planificado = @dFecha_Cierre_Planificado


    INSERT INTO cierreAsistencias
    SELECT
        @nId_Cierre,
        a.nId_Colaborador,
        p.sPersona_Nombre AS sNombre_Colaborador,
        (SELECT sNombre_Supervisor
        FROM dbo.fn_supervisorsName_by_collaborator(a.nId_Colaborador)
        WHERE nIsFinalApprover = 1 ) AS sNombre_Director,
        SUM(a.dTiempo_Laboradas)  AS dCantidad_Horas_Laboradas,
        COUNT(CASE
            WHEN a.nEstado_Asistencia IN (2, 3, 5, 6, 9, 11) OR
                 (a.nEstado_Asistencia IN (10) AND lic.contador > 0) THEN 1
            ELSE NULL
        END) AS nCantidad_Dias_Laborados,
        ((DATEDIFF(DAY, @dFecha_Inicio_Asistencia, @dFecha_Fin_Asistencia) + 1) -
            SUM(CASE
                WHEN a.nEstado_Asistencia = 4 OR
                     (a.nEstado_Asistencia = 10 AND no_lic.contador > 0)
                THEN 1
                ELSE 0
            END)) AS nCantidad_Dias_Laborados_Planilla,
        SUM(a.dTiempo_No_Laboradas)  AS dCantidad_Horas_No_Laboradas,
        SUM(CASE
            WHEN a.nEstado_Asistencia = 4 OR
                 (a.nEstado_Asistencia = 10 AND no_lic.contador > 0)
            THEN 1
            ELSE 0
        END) AS nCantidad_Dias_Faltas,
        (SELECT SUM(dTiempo_Tardanza)
         FROM Asistencias
         WHERE nId_Colaborador = a.nId_Colaborador
         AND dFecha_Asistencia BETWEEN @dFecha_Inicio_Tardanza AND @dFecha_Fin_Tardanza)
         AS dTiempo_Tardanza_Horas,
        SUM(a.dTiempo_Extra)  AS dTiempo_Extra_Horas,
        SUM(a.dTiempo_Avance)  AS dTiempo_Avance_Horas,
        SUM(a.dTiempo_Recuperacion)  AS dTiempo_Recuperacion,
        SUM(a.dTiempo_Compensacion)  AS dTiempo_Compensacion,
        (SUM(a.dTiempo_No_Laboradas) + SUM(a.dTiempo_Tardanza))  AS dCantidad_Descuento_Horas,
        SUM(a.dTiempo_Permisos)  AS dTiempo_Permisos_Horas,
        CONCAT_WS(',',
            (SELECT STRING_AGG(
                CASE
                    WHEN s.dFecha_Inicio != s.dFecha_Fin
                    THEN CONCAT(ts.sDescripcion, ' DEL ', s.dFecha_Inicio, ' AL ', s.dFecha_Fin)
                    ELSE CONCAT(ts.sDescripcion, ' EL ', s.dFecha_Inicio)
                END, ', ')
             FROM Solicitudes s
             JOIN Tipos_Solicitudes ts ON s.nId_Tipo_Solicitud = ts.nId_Tipo_Solicitud
             WHERE ts.nId_Categoria NOT IN (4)
             AND s.nEstado_Solicitud = 3
             AND s.nId_Colaborador = a.nId_Colaborador
             AND ((s.dFecha_Inicio BETWEEN @dFecha_Inicio_Asistencia AND @dFecha_Fin_Asistencia)
             OR (s.dFecha_Fin BETWEEN @dFecha_Inicio_Asistencia AND @dFecha_Fin_Asistencia))),
            (SELECT STRING_AGG(CONCAT('FALTA ', dFecha_Asistencia), ', ')
             FROM Asistencias
             WHERE nId_Colaborador = a.nId_Colaborador
             AND dFecha_Asistencia BETWEEN @dFecha_Inicio_Asistencia AND @dFecha_Fin_Asistencia
             AND nEstado_Asistencia = 4)) AS sDetalle,
        ISNULL(tsm.dTotal_Tardanzas_Sin_Tolerancia, 0),
        ISNULL(tsm.dTotal_Tardanzas_Con_Tolerancia, 0),
        0 AS dTiempo_Tarzanda_Descontado
    FROM Asistencias a
    JOIN Colaboradores c ON c.nId_Colaborador = a.nId_Colaborador
    JOIN Personas p ON p.nId_Persona = c.nId_Persona
    LEFT JOIN
        (SELECT s.nId_Colaborador, COUNT(s.nId_Solictud) AS contador
         FROM Solicitudes s
         JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud
         WHERE ts.nId_Categoria IN (3)
         AND s.nId_Tipo_Solicitud NOT IN (19)
         AND ((s.dFecha_Inicio BETWEEN @dFecha_Inicio_Asistencia AND @dFecha_Fin_Asistencia)
         OR (s.dFecha_Fin BETWEEN @dFecha_Inicio_Asistencia AND @dFecha_Fin_Asistencia))
         GROUP BY nId_Colaborador) AS lic ON lic.nId_Colaborador = a.nId_Colaborador
    LEFT JOIN (
        SELECT
            nId_Colaborador,
            (SUM(CASE WHEN nTipo_Transaccion = 1 AND nId_Sub_Tipo_Entidad = 3 THEN dCantidad_Minutos ELSE 0 END) -
             SUM(CASE WHEN nTipo_Transaccion = 2 AND nId_Sub_Tipo_Entidad = 3 THEN dCantidad_Minutos ELSE 0 END))
             AS dTotal_Tardanzas_Con_Tolerancia,
            (SUM(CASE WHEN nTipo_Transaccion = 1 AND nId_Sub_Tipo_Entidad = 4 THEN dCantidad_Minutos ELSE 0 END) -
             SUM(CASE WHEN nTipo_Transaccion = 2 AND nId_Sub_Tipo_Entidad = 4 THEN dCantidad_Minutos ELSE 0 END))
             AS dTotal_Tardanzas_Sin_Tolerancia
        FROM Transacciones_Saldo_Mins
        WHERE nEstado_Transaccion = 1
        AND nId_Cierre = @nId_Cierre
        GROUP BY nId_Colaborador
    ) AS tsm ON tsm.nId_Colaborador = a.nId_Colaborador
    LEFT JOIN
    (SELECT s.nId_Colaborador, COUNT(s.nId_Solictud) AS contador
     FROM Solicitudes s
     WHERE nId_Tipo_Solicitud = 19
     AND ((dFecha_Inicio BETWEEN @dFecha_Inicio_Asistencia AND @dFecha_Fin_Asistencia)
     OR (dFecha_Fin BETWEEN @dFecha_Inicio_Asistencia AND @dFecha_Fin_Asistencia))
     GROUP BY nId_Colaborador) AS no_lic ON no_lic.nId_Colaborador = a.nId_Colaborador
    WHERE (a.dFecha_Asistencia BETWEEN @dFecha_Inicio_Asistencia AND @dFecha_Fin_Asistencia)
    GROUP BY a.nId_Colaborador,
             p.sPersona_Nombre,
          --   sups.sNombre_Supervisor,
             tsm.dTotal_Tardanzas_Sin_Tolerancia,
             tsm.dTotal_Tardanzas_Con_Tolerancia;


END TRY
BEGIN CATCH
 IF (@@TRANCOUNT > 0)
 ROLLBACK TRANSACtION;

 SELECT ERROR_MESSAGE() AS ErrorMessage,
 ERROR_LINE() AS ErrorLine;
 END CATCH;

 IF (@@TRANCOUNT > 0)
 COMMIT TRANSACTION;