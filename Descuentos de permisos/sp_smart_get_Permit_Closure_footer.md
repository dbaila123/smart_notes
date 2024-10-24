```sql
CREATE PROCEDURE sp_smart_get_Permit_Closure_footer
(
    @nId INT, -- Id Colaborador
    @sSlugToGetList NVARCHAR(MAX),
    @dFecha_Inicio VARCHAR(MAX),
    @dFecha_Fin VARCHAR(MAX),
    @sFilterOne NVARCHAR(MAX) -- Filtro por equipos
)
AS
BEGIN
    DECLARE @viewClause AS NVARCHAR(MAX);
    DECLARE @WhereClause AS NVARCHAR(MAX);
    DECLARE @conditions_array TABLE(condition NVARCHAR(MAX))
    DECLARE @condition AS NVARCHAR(MAX);
    DECLARE @count INT;
    DECLARE @Script AS NVARCHAR(MAX);
    DECLARE @ColaboradoresActivosScript AS NVARCHAR(MAX);
    DECLARE @ColaboradoresCesadosScript AS NVARCHAR(MAX);
    DECLARE @EstadoColaboradorActivo NVARCHAR(MAX) = '1'; -- Activo
    DECLARE @EstadoColaboradorCesado NVARCHAR(MAX) = '2'; -- Cesado

    -- Tipo de listado
    IF @sSlugToGetList = 'REQUEST-CLOSING-LIST'
    BEGIN
        SET @viewClause = N'SELECT * FROM (SELECT * FROM v_listado_Transacciones_Saldo_Mins WHERE nId_Colaborador = ' + CONVERT(VARCHAR, @nId) + ') a';
    END

    -- Filtros
    IF @dFecha_Inicio IS NOT NULL AND @dFecha_Fin IS NOT NULL
    BEGIN
        EXEC sp_get_dates_condition 'dDatetime_Creacion', @dFecha_Inicio, @dFecha_Fin, @condition OUTPUT
        INSERT INTO @conditions_array VALUES(@condition)
    END

    IF @sFilterOne IS NOT NULL
    BEGIN
        SET @condition = 'nId_Colaborador IN (' + @sFilterOne + ')'
        INSERT INTO @conditions_array VALUES(@condition)
    END

    -- Where
    SELECT @count = COUNT(*) FROM @conditions_array;
    IF @count > 0
    BEGIN
        SET @WhereClause = N'WHERE ';
    END

    WHILE @count > 0
    BEGIN
        DECLARE @condition_temp VARCHAR(MAX) = (SELECT TOP(1) condition FROM @conditions_array);
        IF @count = 1
        BEGIN
            SET @WhereClause = CONCAT(@WhereClause, @condition_temp);
        END
        ELSE
        BEGIN
            SET @WhereClause = CONCAT(@WhereClause, @condition_temp, ' AND ');
        END
        DELETE TOP (1) FROM @conditions_array
        SELECT @count = COUNT(*) FROM @conditions_array;
    END

    -- Scripts para conteos
    SET @ColaboradoresActivosScript = 'SELECT COUNT(*) FROM (' + @viewClause + ' ' + @WhereClause + ') a WHERE a.nEstado = ' + @EstadoColaboradorActivo;
    SET @ColaboradoresCesadosScript = 'SELECT COUNT(*) FROM (' + @viewClause + ' ' + @WhereClause + ') a WHERE a.nEstado = ' + @EstadoColaboradorCesado;

    -- Tablas temporales
    CREATE TABLE #ColaboradoresActivosTemp (colaboradoresActivos INT);
    CREATE TABLE #ColaboradoresCesadosTemp (colaboradoresCesados INT);

    -- Insertar datos en tablas
    INSERT INTO #ColaboradoresActivosTemp
    EXEC sp_executesql @ColaboradoresActivosScript;

    INSERT INTO #ColaboradoresCesadosTemp
    EXEC sp_executesql @ColaboradoresCesadosScript;

    -- Seleccionar resultados
    SELECT * FROM #ColaboradoresActivosTemp, #ColaboradoresCesadosTemp;

    -- Eliminar tablas temporales
    DROP TABLE #ColaboradoresActivosTemp;
    DROP TABLE #ColaboradoresCesadosTemp;
END

```