```sql
CREATE OR ALTER PROCEDURE sp_smart_get_Permit_Closure
  
(      
    @nId_Colaborador INT, -- Id Colaborador    
    @SNombreColaborador NVARCHAR(MAX),  
    @sSlugToGetList NVARCHAR(MAX),    
    @dFecha_Fin date, -- Solo la fecha final
    @sFilterOne NVARCHAR
(MAX), -- Filtro por equipos      
    @pnum INT, -- Número de Página      
    @size INT, -- Cantidad de Registros por página      
    @campo VARCHAR(MAX), -- Nombre de columna para ordenar      
    @order VARCHAR(MAX),  -- asc || desc
	@bDownload BIT 
= 0
)      
AS       
BEGIN      
    DECLARE @script NVARCHAR(MAX);      
    DECLARE @viewClause NVARCHAR(MAX);      
    DECLARE @whereClause NVARCHAR(MAX) = '';  -- Inicializar como vacío  
    DECLARE @orderClause NVARCHAR(MAX);      
    DECLARE @paginationClause NVARCHAR(MAX);      
    DECLARE @contador NVARCHAR(MAX);      
  
IF @sSlugToGetList = 'REQUEST-CLOSING-LIST'
BEGIN
    SET @viewClause = CASE
                        WHEN @dFecha_Fin IS NOT NULL
                        THEN N'SELECT * FROM dbo.fn_get_request_hours_by_Transaction(null, 1, 0, 1, NULL, ''' + CONVERT(NVARCHAR, @dFecha_Fin, 120) + ''')'
                        ELSE N'SELECT * FROM v_listado_Transacciones_Saldo_Mins'
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
  
    IF @count
 > 0      
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
  
    -- Orden y Paginación  

    EXEC sp_get_order_clause @campo, @order , 'FechaUltimaTransaccion' , @orderClause OUTPUT;   
    EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT;      
  
--  IF @bDownload = 1
--  BEGIN
    DECLARE @total INT = 100;      
    DECLARE @ParmDefinition NVARCHAR(MAX);      
  
    SET @contador = CONCAT('SELECT @totalOut = COUNT(*) FROM (', @viewClause, ' ', @whereClause, ') a');      
    SET @ParmDefinition = N'@totalOut INT OUTPUT';       
    EXEC sp_executesql @contador, @ParmDefinition, @totalOut = @total OUTPUT;      
  
    BEGIN  
        SET @script = CONCAT('SELECT *, ', @total, ' AS nTotal FROM (', @viewClause, ' ', @whereClause, ') a ', @orderClause, ' ', @paginationClause);      
    END  
  -- END
    PRINT @script;  
    
    EXECUTE sp_executesql @script;      
END
```
