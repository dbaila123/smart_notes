```sql
create or alter PROCEDURE sp_smart_get_solicitudes_cierre_Footer
(    
    @nId_Colaborador INT, -- Id Colaborador  
    @SNombreColaborador NVARCHAR(MAX),
    @sSlugToGetList NVARCHAR(MAX),  
    
	@dFecha_Fin VARCHAR(MAX), -- Solo la fecha final
    @sFilterOne NVARCHAR(MAX)--, -- Filtro por equipos    
)
AS     
BEGIN    
    DECLARE @script NVARCHAR(MAX);    
    DECLARE @viewClause NVARCHAR(MAX);    
    DECLARE @whereClause NVARCHAR(MAX) = '';  -- Inicializar como vacío
    DECLARE @orderClause NVARCHAR(MAX);    
    DECLARE @paginationClause NVARCHAR(MAX);    
    DECLARE @contador NVARCHAR(MAX);    

	declare @totalHoras nvarchar(max);
	declare @totalRecuperado nvarchar(max);
	declare @totalNoRecuperado nvarchar(max);

    IF @sSlugToGetList = 'REQUEST-CLOSING-LIST'    
    BEGIN    
		-- SET @viewClause = N'SELECT * FROM v_listado_Transacciones_Saldo_Mins';    
		SET @viewClause = N'SELECT nId_colaborador, Snombrecolaborador, total_solicitudes, cantidadtotalsuma, cantidadtotalresta, cantidadtotaldescontado, fechaultimatransaccion, surl_foto FROM v_listado_Transacciones_Saldo_Mins';
    END        

    DECLARE @conditions_array TABLE(condition NVARCHAR(MAX));    
    DECLARE @condition NVARCHAR(MAX);

    -- Filtro por Colaborador
    --IF @nId_Colaborador IS NOT NULL AND @nId_Colaborador <> 0
    --BEGIN
        --IF @whereClause = N'' 
            --SET @whereClause += N' WHERE nId_Colaborador = ' + CONVERT(VARCHAR, @nId_Colaborador);
        --ELSE 
            --SET @whereClause += N' AND nId_Colaborador = ' + CONVERT(VARCHAR, @nId_Colaborador);
    --END

    -- Filtro por Nombre de Colaborador
    IF @SNombreColaborador IS NOT NULL  and @SNombreColaborador <> ''
    BEGIN
        EXEC sp_get_like_condition 'SNombreColaborador', @SNombreColaborador, @condition OUTPUT
        INSERT INTO @conditions_array VALUES(@condition)
    END

    -- Filtro por Fecha
    IF @dFecha_Fin IS NOT NULL    
    BEGIN
        IF @whereClause = N'' 
            SET @whereClause += N' WHERE FechaUltimaTransaccion <= ''' + @dFecha_Fin + '''';    
        ELSE 
            SET @whereClause += N' AND FechaUltimaTransaccion <= ''' + @dFecha_Fin + '''';    
    END  

    -- Filtro por Equipos
    IF @sFilterOne IS NOT NULL 
    BEGIN    
        DECLARE @CollaboratorsListTeam NVARCHAR(MAX);       
        SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(@sFilterOne, 0, 0);     
        SET @condition = 'nId_Colaborador IN (' + @CollaboratorsListTeam + ')';    
        INSERT INTO @conditions_array VALUES(@condition);    
    END

    DECLARE @count INT;    
    SELECT @count = COUNT(*) FROM @conditions_array;

    IF @count > 0    
    BEGIN    
        IF @whereClause = N'' 
            SET @whereClause = N' WHERE ';    
        ELSE 
            SET @whereClause += N' AND ';    
    END 
    
    -- Combina todas las condiciones
    WHILE @count > 0    
    BEGIN    
        DECLARE @condition_temp NVARCHAR(MAX) = (SELECT TOP(1) condition FROM @conditions_array);    
        IF @count = 1    
        BEGIN
            SET @whereClause += @condition_temp;    
        END
        ELSE
        BEGIN
            SET @whereClause += @condition_temp + ' AND ';    
        END    
        DELETE TOP (1) FROM @conditions_array;    
        SELECT @count = COUNT(*) FROM @conditions_array;    
    END

	-- david:
	SET @totalHoras =  concat (@viewClause,' ',@whereClause)
 --   SET @totalRecuperado =  concat (@viewClause,' ',@whereClause, ' and nEstado in('+@EstadoPendiente+') and nId_Tipo_Solicitud in('+ @TipoSolicitudesHora+')');
 --  	SET @totalNoRecuperado =  concat (@viewClause,' ',@whereClause, ' and nEstado in('+@EstadoAprobado+') and nId_Tipo_Solicitud in('+ @TipoSolicitudesDia+')');



    -- Orden y Paginación
    -- EXEC sp_get_order_clause @campo, @order , 'FechaUltimaTransaccion' , @orderClause OUTPUT; 
    -- EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT;    

    DECLARE @total INT = 100;    
    DECLARE @ParmDefinition NVARCHAR(MAX);    

    SET @contador = CONCAT('SELECT @totalOut = COUNT(*) FROM (', @viewClause, ' ', @whereClause, ') a');    
    SET @ParmDefinition = N'@totalOut INT OUTPUT';     
    EXEC sp_executesql @contador, @ParmDefinition, @totalOut = @total OUTPUT;    

    BEGIN
        SET @script = CONCAT('SELECT *, ', @total, ' AS nTotal FROM (', @viewClause, ' ', @whereClause, ') a ', @orderClause, ' ', @paginationClause);    
    END

    --PRINT @script;    
    -- EXECUTE sp_executesql @script;    


	CREATE TABLE #TotalHorasTemp (
		nid_Colaborador int,
		sNombreColaborador varchar(1000),
		total_solicitudes int,
		cantidadtotalsuma float,
		cantidadtotalresta float,
		cantidadtotaldescontado float,
		fechaultimatransaccion date,
		surl_foto varchar(1000)
	)
	insert into #TotalHorasTemp 
	EXEC sp_executesql @totalHoras

	select @totalHoras  = sum(CANTIDADTOTALSUMA) from #TotalHorasTemp
	select @totalRecuperado = sum(CANTIDADTOTALRESTA) from #TotalHorasTemp
	select @totalNoRecuperado = sum(CANTIDADTOTALDESCONTADO) from #TotalHorasTemp

	select
		@totalHoras totalHours,
		@totalRecuperado totalHoursRecovered,
		@totalNoRecuperado totalHoursNotRecovered
	DROP TABLE #TotalHorasTemp
END
go
```

```sql
create or alter PROCEDURE sp_smart_get_solicitudes_cierre_Footer
(        
    @nId_Colaborador INT, -- Id Colaborador      
    @SNombreColaborador NVARCHAR(MAX),    
    @sSlugToGetList NVARCHAR(MAX),      
        
 @dFecha_Fin VARCHAR(MAX), -- Solo la fecha final    
    @sFilterOne NVARCHAR(MAX) -- Filtro por equipos    
)    
AS         
BEGIN        
    DECLARE @script NVARCHAR(MAX);        
    DECLARE @viewClause NVARCHAR(MAX);        
    DECLARE @whereClause NVARCHAR(MAX) = '';  -- Inicializar como vacío    
    DECLARE @orderClause NVARCHAR(MAX);        
    DECLARE @paginationClause NVARCHAR(MAX);        
    DECLARE @contador NVARCHAR(MAX);        
    
 declare @totalHoras nvarchar(max);    
 declare @totalRecuperado nvarchar(max);    
 declare @totalNoRecuperado nvarchar(max);    
    
    IF @sSlugToGetList = 'REQUEST-CLOSING-LIST'        
    BEGIN
        SET @viewClause = CASE
                            WHEN @dFecha_Fin IS NOT NULL
                            THEN N'SELECT  nId_colaborador, Snombrecolaborador, total_solicitudes, cantidadtotalsuma, cantidadtotalresta, cantidadtotaldescontado, fechaultimatransaccion, surl_foto  FROM dbo.fn_get_request_hours_by_Transaction(null, 1, 0, 1, NULL, ''' + CONVERT(NVARCHAR, @dFecha_Fin, 120) + ''')'
                            ELSE N'SELECT  nId_colaborador, Snombrecolaborador, total_solicitudes, cantidadtotalsuma, cantidadtotalresta, cantidadtotaldescontado, fechaultimatransaccion, surl_foto  FROM v_listado_Transacciones_Saldo_Mins'
                          END;
    
    END            
    
    DECLARE @conditions_array TABLE(condition NVARCHAR(MAX));        
    DECLARE @condition NVARCHAR(MAX);    
    
    -- Filtro por Colaborador    
    --IF @nId_Colaborador IS NOT NULL AND @nId_Colaborador <> 0    
    --BEGIN    
        --IF @whereClause = N''     
            --SET @whereClause += N' WHERE nId_Colaborador = ' + CONVERT(VARCHAR, @nId_Colaborador);    
        --ELSE     
            --SET @whereClause += N' AND nId_Colaborador = ' + CONVERT(VARCHAR, @nId_Colaborador);    
    --END    
    
    -- Filtro por Nombre de Colaborador    
    IF @SNombreColaborador IS NOT NULL  and @SNombreColaborador <> ''    
    BEGIN    
        EXEC sp_get_like_condition 'SNombreColaborador', @SNombreColaborador, @condition OUTPUT    
        INSERT INTO @conditions_array VALUES(@condition)    
    END    
    
    -- Filtro por Fecha    
    IF @dFecha_Fin IS NOT NULL        
    BEGIN    
        IF @whereClause = N''     
            SET @whereClause += N' WHERE FechaUltimaTransaccion <= ''' + @dFecha_Fin + '''';        
        ELSE     
            SET @whereClause += N' AND FechaUltimaTransaccion <= ''' + @dFecha_Fin + '''';        
    END      
    
    -- Filtro por Equipos    
    IF @sFilterOne IS NOT NULL     
    BEGIN        
        DECLARE @CollaboratorsListTeam NVARCHAR(MAX);           
        SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(@sFilterOne, 0, 0);         
        SET @condition = 'nId_Colaborador IN (' + @CollaboratorsListTeam + ')';        
        INSERT INTO @conditions_array VALUES(@condition);        
    END    
    
    DECLARE @count INT;        
    SELECT @count = COUNT(*) FROM @conditions_array;    
    
    IF @count > 0        
    BEGIN        
        IF @whereClause = N''     
            SET @whereClause = N' WHERE ';        
        ELSE     
            SET @whereClause += N' AND ';        
    END     
        
    -- Combina todas las condiciones    
    WHILE @count > 0        
    BEGIN        
        DECLARE @condition_temp NVARCHAR(MAX) = (SELECT TOP(1) condition FROM @conditions_array);        
        IF @count = 1        
        BEGIN    
            SET @whereClause += @condition_temp;        
        END    
        ELSE    
        BEGIN    
            SET @whereClause += @condition_temp + ' AND ';        
        END        
        DELETE TOP (1) FROM @conditions_array;        
        SELECT @count = COUNT(*) FROM @conditions_array;        
    END    
    
 -- david:    
 SET @totalHoras =  concat (@viewClause,' ',@whereClause)    
 --   SET @totalRecuperado =  concat (@viewClause,' ',@whereClause, ' and nEstado in('+@EstadoPendiente+') and nId_Tipo_Solicitud in('+ @TipoSolicitudesHora+')');    
 --   SET @totalNoRecuperado =  concat (@viewClause,' ',@whereClause, ' and nEstado in('+@EstadoAprobado+') and nId_Tipo_Solicitud in('+ @TipoSolicitudesDia+')');    
    
    
    
    -- Orden y Paginación    
    -- EXEC sp_get_order_clause @campo, @order , 'FechaUltimaTransaccion' , @orderClause OUTPUT;     
    -- EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT;         
    DECLARE @total INT = 100;        
    DECLARE @ParmDefinition NVARCHAR(MAX);        
    
    SET @contador = CONCAT('SELECT @totalOut = COUNT(*) FROM (', @viewClause, ' ', @whereClause, ') a');        
    SET @ParmDefinition = N'@totalOut INT OUTPUT';         
    EXEC sp_executesql @contador, @ParmDefinition, @totalOut = @total OUTPUT;        
    
    BEGIN    
        SET @script = CONCAT('SELECT *, ', @total, ' AS nTotal FROM (', @viewClause, ' ', @whereClause, ') a ', @orderClause, ' ', @paginationClause);        
    END     
    --PRINT @script;        
    -- EXECUTE sp_executesql @script;        
    
    
 CREATE TABLE #TotalHorasTemp (    
  nid_Colaborador int,    
  sNombreColaborador varchar(1000),    
  total_solicitudes int,    
  cantidadtotalsuma float,    
  cantidadtotalresta float,    
  cantidadtotaldescontado float,    
  fechaultimatransaccion date,    
  surl_foto varchar(1000)    
 )    
 insert into #TotalHorasTemp     
 EXEC sp_executesql @totalHoras    
    
 select @totalHoras  = sum(CANTIDADTOTALSUMA) from #TotalHorasTemp    
 select @totalRecuperado = sum(CANTIDADTOTALRESTA) from #TotalHorasTemp    
 select @totalNoRecuperado = sum(CANTIDADTOTALDESCONTADO) from #TotalHorasTemp    
    
 select    
  @totalHoras totalHours,    
  @totalRecuperado totalHoursRecovered,    
  @totalNoRecuperado totalHoursNotRecovered    
 DROP TABLE #TotalHorasTemp    
END
go
```