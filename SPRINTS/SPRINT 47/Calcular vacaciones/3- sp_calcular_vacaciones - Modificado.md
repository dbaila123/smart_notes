```SQL
CREATE PROCEDURE [dbo].[sp_calcular_vacaciones]
	@dFecha DATETIME,
    @nUsuario_Update int
AS
BEGIN
    SET NOCOUNT ON;
    
    CREATE TABLE #calculo_vacaciones (
        nId_Colaborador int PRIMARY KEY,
        dfec_ingreso_real datetime,
        nId_Regimen int,
        nVacaciones_Dias_Calendario int,
        nVacaciones_Dias_Laborales int,
        -- Fecha límite para el cálculo (fecha de cese o @dFecha)
        dFecha_Limite_Calculo datetime,
        -- Cálculos base
        nMeses_Trabajados int,
        nAnios_Completos int,
        dProximo_Aniversario_Anual datetime,
        -- Días generados (acumulado mensual que no se ha convertido en ganado)
        dDias_Generados_Laborales decimal(18,16),
        dDias_Generados_Calendario decimal(18,6),
        -- Días ganados (disponibles del periodo actual)
        dDias_Ganados_Laborales decimal(18,6),
        dDias_Ganados_Calendario decimal(18,6),
        -- Días vencidos (de periodos anteriores)
        dDias_Vencidos_Laborales decimal(18,6),
        dDias_Vencidos_Calendario decimal(18,6),
        -- Días tomados
        dDias_Tomados_Total_Laborales decimal(18,6),
        dDias_Tomados_Vencidos_Laborales decimal(18,6),
        dDias_Tomados_Vigente_Laborales decimal(18,6),
        dDias_Tomados_Total_Calendario decimal(18,6),
        dDias_Tomados_Vencidos_Calendario decimal(18,6),
        dDias_Tomados_Vigente_Calendario decimal(18,6),
        -- Último periodo ganado
        dUltimo_Periodo_Ganado datetime
    );

    -- FASE 1: Obtener información base de todos los colaboradores activos Y cesados
    INSERT INTO #calculo_vacaciones (
        nId_Colaborador, dfec_ingreso_real, nId_Regimen, 
        nVacaciones_Dias_Calendario, nVacaciones_Dias_Laborales,
        dFecha_Limite_Calculo,
        nMeses_Trabajados, nAnios_Completos, dProximo_Aniversario_Anual,
        dDias_Generados_Laborales, dDias_Generados_Calendario,
        dDias_Ganados_Laborales, dDias_Ganados_Calendario,
        dDias_Vencidos_Laborales, dDias_Vencidos_Calendario,
        dDias_Tomados_Total_Laborales, dDias_Tomados_Vencidos_Laborales, 
        dDias_Tomados_Vigente_Laborales, dDias_Tomados_Total_Calendario,
        dDias_Tomados_Vencidos_Calendario, dDias_Tomados_Vigente_Calendario,
        dUltimo_Periodo_Ganado
    )
    SELECT 
        c.nId_Colaborador,
        c.dfec_ingreso_real,
        r.nId_Regimen,
        r.nVacaciones_Dias_Calendario,
        r.nVacaciones_Dias_Laborales,
        -- Fecha límite: si está cesado, usar fecha de cese; si no, usar @dFecha
        CASE 
            WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
            THEN c.dDatetime_Delete
            ELSE @dFecha
        END AS dFecha_Limite_Calculo,
        -- Calcular meses y años trabajados hasta la fecha límite
        DATEDIFF(MONTH, c.dfec_ingreso_real, 
            CASE 
                WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
                THEN c.dDatetime_Delete
                ELSE @dFecha
            END)
        - CASE 
            WHEN DAY(CASE 
                        WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
                        THEN c.dDatetime_Delete
                        ELSE @dFecha
                     END) < DAY(c.dfec_ingreso_real)
            THEN 1 
            ELSE 0 
          END AS nMeses_Trabajados,
        DATEDIFF(YEAR, c.dfec_ingreso_real, 
            CASE 
                WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
                THEN c.dDatetime_Delete
                ELSE @dFecha
            END) 
        - CASE 
            WHEN DATEADD(YEAR, DATEDIFF(YEAR, c.dfec_ingreso_real, 
                              CASE 
                                  WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
                                  THEN c.dDatetime_Delete
                                  ELSE @dFecha
                              END), c.dfec_ingreso_real) > 
                 CASE 
                     WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
                     THEN c.dDatetime_Delete
                     ELSE @dFecha
                 END
            THEN 1 
            ELSE 0 
          END AS nAnios_Completos,
        -- Próximo aniversario anual (considerando fecha límite)
        CASE 
            WHEN DATEADD(YEAR, DATEDIFF(YEAR, c.dfec_ingreso_real, 
                              CASE 
                                  WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
                                  THEN c.dDatetime_Delete
                                  ELSE @dFecha
                              END), c.dfec_ingreso_real) <= 
                 CAST(CASE 
                         WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
                         THEN c.dDatetime_Delete
                         ELSE @dFecha
                      END AS DATE)
            THEN DATEADD(YEAR, DATEDIFF(YEAR, c.dfec_ingreso_real, 
                               CASE 
                                   WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
                                   THEN c.dDatetime_Delete
                                   ELSE @dFecha
                               END) + 1, c.dfec_ingreso_real)
            ELSE DATEADD(YEAR, DATEDIFF(YEAR, c.dfec_ingreso_real, 
                               CASE 
                                   WHEN c.dDatetime_Delete IS NOT NULL AND c.dDatetime_Delete < @dFecha 
                                   THEN c.dDatetime_Delete
                                   ELSE @dFecha
                               END), c.dfec_ingreso_real)
        END AS dProximo_Aniversario_Anual,
        -- Inicializar en 0
        0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
        NULL
    FROM Colaboradores c
    INNER JOIN Empresas_Usuarias eu ON c.nId_Empresa = eu.nId_Empresa
    INNER JOIN Regimen r ON eu.nId_Regimen = r.nId_Regimen
    WHERE c.dfec_ingreso_real <= @dFecha -- Solo colaboradores que ya ingresaron
        AND (c.dDatetime_Delete IS NULL OR c.dDatetime_Delete <= @dFecha); -- Incluir activos y cesados hasta @dFecha

    -- FASE 2: Calcular días generados y ganados
    UPDATE #calculo_vacaciones
    SET 
        -- Días generados = meses del año actual * (días anuales / 12)
        -- Solo los meses que no han completado un año
        dDias_Generados_Laborales = 
            CASE 
                WHEN nAnios_Completos = 0 THEN nMeses_Trabajados * (CAST(nVacaciones_Dias_Laborales AS decimal(18,16)) / 12.0)
                ELSE (nMeses_Trabajados % 12) * (CAST(nVacaciones_Dias_Laborales AS decimal(18,16)) / 12.0)
            END,
        dDias_Generados_Calendario = 
            CASE 
                WHEN nAnios_Completos = 0 THEN nMeses_Trabajados * (CAST(nVacaciones_Dias_Calendario AS decimal(18,6)) / 12.0)
                ELSE (nMeses_Trabajados % 12) * (CAST(nVacaciones_Dias_Calendario AS decimal(18,6)) / 12.0)
            END,
        -- Días ganados = años completos * días anuales
        dDias_Ganados_Laborales = nAnios_Completos * CAST(nVacaciones_Dias_Laborales AS decimal(18,6)),
        dDias_Ganados_Calendario = nAnios_Completos * CAST(nVacaciones_Dias_Calendario AS decimal(18,6)),
        -- Último periodo ganado
        dUltimo_Periodo_Ganado = DATEADD(YEAR, nAnios_Completos, dfec_ingreso_real);

    -- FASE 3: Calcular días tomados de las solicitudes (considerando fecha límite)
    WITH dias_tomados AS (
        SELECT 
            s.nId_Colaborador,
            SUM(s.nCantidad) as total_dias_laborales,
            -- Convertir días laborales a calendario (aproximación: laborales * 7/5)
            SUM(s.nCantidad * 7.0 / 5.0) as total_dias_calendario
        FROM Solicitudes s
        INNER JOIN Tipos_Solicitudes ts ON s.nId_Tipo_Solicitud = ts.nId_Tipo_Solicitud
        INNER JOIN #calculo_vacaciones cv ON s.nId_Colaborador = cv.nId_Colaborador
        WHERE s.nEstado_Solicitud IN (1, 3, 4) -- Pendiente, Aprobado, Pendiente Director
            AND ts.nId_Tipo_Solicitud IN (3, 10, 22) -- Reserva, Venta, Adelanto
            AND s.nEstado = 1 -- Activo
            --AND s.dFecha_Inicio <= cv.dFecha_Limite_Calculo -- Solo solicitudes hasta la fecha límite del colaborador
        GROUP BY s.nId_Colaborador
    )
    UPDATE cv
    SET 
        cv.dDias_Tomados_Total_Laborales = ISNULL(dt.total_dias_laborales, 0),
        cv.dDias_Tomados_Total_Calendario = ISNULL(dt.total_dias_calendario, 0)
    FROM #calculo_vacaciones cv
    LEFT JOIN dias_tomados dt ON cv.nId_Colaborador = dt.nId_Colaborador;

    -- FASE 4: Calcular días vencidos y distribuir días tomados
    -- Los días vencidos son todos los días ganados menos los tomados
    UPDATE #calculo_vacaciones
    SET 
        -- Para días laborales
        dDias_Vencidos_Laborales = 
            CASE 
                WHEN dDias_Ganados_Laborales > dDias_Tomados_Total_Laborales 
                THEN dDias_Ganados_Laborales - dDias_Tomados_Total_Laborales
                ELSE 0
            END,
        dDias_Tomados_Vencidos_Laborales = 
            CASE 
                WHEN dDias_Tomados_Total_Laborales > dDias_Ganados_Laborales 
                THEN dDias_Ganados_Laborales
                ELSE dDias_Tomados_Total_Laborales
            END,
        dDias_Tomados_Vigente_Laborales = 
            CASE 
                WHEN dDias_Tomados_Total_Laborales > dDias_Ganados_Laborales 
                THEN dDias_Tomados_Total_Laborales - dDias_Ganados_Laborales
                ELSE 0
            END,
        -- Para días calendario
        dDias_Vencidos_Calendario = 
            CASE 
                WHEN dDias_Ganados_Calendario > dDias_Tomados_Total_Calendario 
                THEN dDias_Ganados_Calendario - dDias_Tomados_Total_Calendario
                ELSE 0
            END,
        dDias_Tomados_Vencidos_Calendario = 
            CASE 
                WHEN dDias_Tomados_Total_Calendario > dDias_Ganados_Calendario 
                THEN dDias_Ganados_Calendario
                ELSE dDias_Tomados_Total_Calendario
            END,
        dDias_Tomados_Vigente_Calendario = 
            CASE 
                WHEN dDias_Tomados_Total_Calendario > dDias_Ganados_Calendario 
                THEN dDias_Tomados_Total_Calendario - dDias_Ganados_Calendario
                ELSE 0
            END;

    UPDATE #calculo_vacaciones
    SET 
        -- Si se tomaron días del periodo vigente, reducir los generados
        dDias_Generados_Laborales = 
            CASE 
                WHEN dDias_Tomados_Vigente_Laborales > 0 AND dDias_Generados_Laborales > dDias_Tomados_Vigente_Laborales
                THEN dDias_Generados_Laborales - dDias_Tomados_Vigente_Laborales
                WHEN dDias_Tomados_Vigente_Laborales > 0 
                THEN 0
                ELSE dDias_Generados_Laborales
            END,
        dDias_Generados_Calendario = 
            CASE 
                WHEN dDias_Tomados_Vigente_Calendario > 0 AND dDias_Generados_Calendario > dDias_Tomados_Vigente_Calendario
                THEN dDias_Generados_Calendario - dDias_Tomados_Vigente_Calendario
                WHEN dDias_Tomados_Vigente_Calendario > 0 
                THEN 0
                ELSE dDias_Generados_Calendario
            END;

    -- FASE 5: Guardar estado anterior para el log
    DECLARE @saldos_anteriores TABLE (
        nId_Colaborador int,
        dDias_Vencidos_Laborales_old decimal(18,6),
        dDias_Ganados_Laborales_old decimal(18,6),
        dDias_Generados_Laborales_old decimal(18,16),
        dDias_Vencidos_Calendario_old decimal(18,6),
        dDias_Ganados_Calendario_old decimal(18,6),
        dDias_Generados_Calendario_old decimal(18,6),
        existe_registro bit
    );

    INSERT INTO @saldos_anteriores
    SELECT 
        cv.nId_Colaborador,
        ISNULL(svc.dDias_Vencidos_Laborales, 0),
        ISNULL(svc.dDias_Ganados_Laborales, 0),
        ISNULL(svc.dDias_Generados_Laborales, 0),
        ISNULL(svc.dDias_Vencidos_Calendario, 0),
        ISNULL(svc.dDias_Ganados_Calendario, 0),
        ISNULL(svc.dDias_Generados_Calendario, 0),
        CASE WHEN svc.nId IS NOT NULL THEN 1 ELSE 0 END
    FROM #calculo_vacaciones cv
    LEFT JOIN saldos_Vacaciones_Colaboradores svc ON cv.nId_Colaborador = svc.nId_Colaborador;

    -- FASE 6: MERGE para insertar o actualizar
    MERGE saldos_Vacaciones_Colaboradores AS target
    USING #calculo_vacaciones AS source
    ON target.nId_Colaborador = source.nId_Colaborador
    WHEN MATCHED THEN
        UPDATE SET
            dDias_Vencidos_Laborales = source.dDias_Vencidos_Laborales,
            dDias_Ganados_Laborales = source.dDias_Generados_Laborales,
            dDias_Generados_Laborales = source.dDias_Generados_Laborales,
            dDias_Vencidos_Calendario = source.dDias_Vencidos_Calendario,
            dDias_Ganados_Calendario = source.dDias_Generados_Calendario,
            dDias_Generados_Calendario = source.dDias_Generados_Calendario,
            nDias_Tomados_Vencidos_Laborales = source.dDias_Tomados_Vencidos_Laborales,
            nDias_Tomados_Vigente_Laborales = source.dDias_Tomados_Vigente_Laborales,
            nDias_Tomados_Vencidos_Calendario = source.dDias_Tomados_Vencidos_Calendario,
            nDias_Tomados_Vigente_Calendario = source.dDias_Tomados_Vigente_Calendario,
            dUltimo_Periodo_Ganado = source.dUltimo_Periodo_Ganado,
            nUsuario_Update = @nUsuario_Update,
            dDatetime_Update = GETDATE()
    WHEN NOT MATCHED THEN
        INSERT (
            nId_Colaborador, dDias_Vencidos_Laborales, dDias_Ganados_Laborales, 
            dDias_Generados_Laborales, dDias_Vencidos_Calendario, dDias_Ganados_Calendario,
            dDias_Generados_Calendario, nDias_Tomados_Vencidos_Laborales, nDias_Tomados_Vigente_Laborales,
            nDias_Tomados_Vencidos_Calendario, nDias_Tomados_Vigente_Calendario,
            dUltimo_Periodo_Ganado, nUsuario_Creador, dDateTime_Creador
        )
        VALUES (
            source.nId_Colaborador, source.dDias_Vencidos_Laborales, source.dDias_Generados_Laborales,
            source.dDias_Generados_Laborales, source.dDias_Vencidos_Calendario, source.dDias_Generados_Calendario,
            source.dDias_Generados_Calendario, source.dDias_Tomados_Vencidos_Laborales, source.dDias_Tomados_Vigente_Laborales,
            source.dDias_Tomados_Vencidos_Calendario, source.dDias_Tomados_Vigente_Calendario,
            source.dUltimo_Periodo_Ganado, @nUsuario_Update, GETDATE()
        );

    -- FASE 7: Registrar cambios en el log
    DECLARE @cambios TABLE (
        nId_Colaborador int,
        sColumna nvarchar(100),
        dValor_Anterior decimal(18,6),
        dValor_Nuevo decimal(18,6)
    );

    -- Comparar y registrar cambios
    INSERT INTO @cambios
    SELECT 
        cv.nId_Colaborador,
        'Dias_Vencidos_Laborales',
        sa.dDias_Vencidos_Laborales_old,
        cv.dDias_Vencidos_Laborales
    FROM #calculo_vacaciones cv
    INNER JOIN @saldos_anteriores sa ON cv.nId_Colaborador = sa.nId_Colaborador
    WHERE sa.dDias_Vencidos_Laborales_old != cv.dDias_Vencidos_Laborales
    
    UNION ALL
    
    SELECT 
        cv.nId_Colaborador,
        'Dias_Ganados_Laborales',
        sa.dDias_Ganados_Laborales_old,
        cv.dDias_Generados_Laborales
    FROM #calculo_vacaciones cv
    INNER JOIN @saldos_anteriores sa ON cv.nId_Colaborador = sa.nId_Colaborador
    WHERE sa.dDias_Ganados_Laborales_old != cv.dDias_Generados_Laborales
    
    UNION ALL
    
    SELECT 
        cv.nId_Colaborador,
        'Dias_Generados_Laborales',
        sa.dDias_Generados_Laborales_old,
        cv.dDias_Generados_Laborales
    FROM #calculo_vacaciones cv
    INNER JOIN @saldos_anteriores sa ON cv.nId_Colaborador = sa.nId_Colaborador
    WHERE sa.dDias_Generados_Laborales_old != cv.dDias_Generados_Laborales
    
    UNION ALL
    
    SELECT 
        cv.nId_Colaborador,
        'Dias_Vencidos_Calendario',
        sa.dDias_Vencidos_Calendario_old,
        cv.dDias_Vencidos_Calendario
    FROM #calculo_vacaciones cv
    INNER JOIN @saldos_anteriores sa ON cv.nId_Colaborador = sa.nId_Colaborador
    WHERE sa.dDias_Vencidos_Calendario_old != cv.dDias_Vencidos_Calendario
    
    UNION ALL
    
    SELECT 
        cv.nId_Colaborador,
        'Dias_Ganados_Calendario',
        sa.dDias_Ganados_Calendario_old,
        cv.dDias_Generados_Calendario
    FROM #calculo_vacaciones cv
    INNER JOIN @saldos_anteriores sa ON cv.nId_Colaborador = sa.nId_Colaborador
    WHERE sa.dDias_Ganados_Calendario_old != cv.dDias_Generados_Calendario
    
    UNION ALL
    
    SELECT 
        cv.nId_Colaborador,
        'Dias_Generados_Calendario',
        sa.dDias_Generados_Calendario_old,
        cv.dDias_Generados_Calendario
    FROM #calculo_vacaciones cv
    INNER JOIN @saldos_anteriores sa ON cv.nId_Colaborador = sa.nId_Colaborador
    WHERE sa.dDias_Generados_Calendario_old != cv.dDias_Generados_Calendario;

    -- Insertar en el log
    INSERT INTO Tb_log_procedure_changes_test (
        sNombre_Procedure, sDetalle_Accion, sNombre_Parametros, sValor_Parametros,
        nUsuario_Create, dDatetime_Create, nUsuario_Update, dDatetime_Update,
        nUsuario_Delete, dDatetime_Delete, sValor_Previo
    )
    SELECT 
        'sp_calcular_vacaciones_v2',
        CONCAT('Los ', c.sColumna, ' del colaborador con id ', c.nId_Colaborador, ' cambió de ', c.dValor_Anterior, ' a ', c.dValor_Nuevo),
        '@dFecha,@nUsuario_Update',
        CONCAT(CONVERT(varchar, @dFecha, 120), ',', @nUsuario_Update),
        @nUsuario_Update,
        GETDATE(),
        NULL, NULL, NULL, NULL,
        (
            SELECT 
                sa.dDias_Vencidos_Laborales_old,
                sa.dDias_Ganados_Laborales_old,
                sa.dDias_Generados_Laborales_old,
                sa.dDias_Vencidos_Calendario_old,
                sa.dDias_Ganados_Calendario_old,
                sa.dDias_Generados_Calendario_old
            FROM @saldos_anteriores sa
            WHERE sa.nId_Colaborador = c.nId_Colaborador
            FOR JSON PATH, WITHOUT_ARRAY_WRAPPER
        )
    FROM @cambios c;

    -- Retornar resumen de cambios (antes de limpiar la tabla temporal)
    SELECT 
        COUNT(DISTINCT c.nId_Colaborador) as Colaboradores_Actualizados,
        COUNT(*) as Total_Cambios_Registrados,
        COUNT(DISTINCT CASE WHEN cv.dFecha_Limite_Calculo < @dFecha THEN cv.nId_Colaborador END) as Colaboradores_Cesados_Actualizados
    FROM @cambios c
    LEFT JOIN #calculo_vacaciones cv ON c.nId_Colaborador = cv.nId_Colaborador;

    -- Limpiar tabla temporal
    DROP TABLE #calculo_vacaciones;
END;