```SQL
CREATE PROCEDURE[dbo].[sp_smart_get_requerimientos_refact]

(

@nId_User int,

@Id_Proyecto int,

@nTipoFilter int, -- Nuevo parámetro: 1 = buscar por nombre proyecto, 2 = buscar por nombre requerimiento

@sNombreRequerimiento nvarchar(max),

@sFilterNotification NVARCHAR(MAX), --Filtro por Id de requerimiento

@fechaCreaIni varchar(max),

@fechaCreaFin varchar(max),

----------------------------------------------------

@campo VARCHAR(MAX), --obligatorio column name toOrder

@order VARCHAR(MAX) --obligatorio asc || desc

)

AS

BEGIN

DECLARE @script AS nvarchar(max);

DECLARE @viewClause AS nvarchar(max);

DECLARE @whereClause AS nvarchar(max);

DECLARE @orderClause AS nvarchar(max);

DECLARE @paginationClause AS nvarchar(max);

DECLARE @contador AS nvarchar(max);

  

DECLARE @nId_Rol int = (select nId_Rol from Roles_Usuarios ru where ru.nId_Usuario = @nId_User);

  

DECLARE @sSlug1 int = (select count(o.sSlug) from Permisos_Opciones po

join Opciones o on o.nId_Opcion = po.nId_Opcion

where po.nId_Rol = @nId_Rol and o.sSlug = 'REQUIREMENTS-LIST-ALL'

and po.nEstado = 1);

  

  

IF @Id_Proyecto = 0 ----reporte

BEGIN

  

IF @sSlug1 >0

BEGIN

SET @viewClause = N'SELECT * FROM v_Listado_Requerimientos_refact'

END

ELSE

---------------------------------------------------------

---------------- PARCHE REVISEN CARACHO -----------------

---------------- QUIERO DORMIR --------------------------

--------------ATTE: VITO --------------------------------

IF @sFilterNotification is not null

begin

--nId_Requerimiento =97055

set @viewClause = N'SELECT * FROM (SELECT * FROM v_Listado_Requerimientos_refact where nId_Requerimiento ='+@sFilterNotification+') r'

end

Else

BEGIN

DECLARE @Requerimientos_a_mostrar TABLE(nId_Requerimiento int)

--Requerimientos a mostrar segun los proyectos

INSERT INTO @Requerimientos_a_mostrar

SELECT

DISTINCT

t.nId_Requerimiento

FROM Requerimientos t

JOIN Proyectos p

ON p.nId_Proyecto = t.nId_Proyecto

JOIN Colaboradores c_lider

ON p.nId_Lider = c_lider.nId_Colaborador

JOIN Personas p_lider

ON c_lider.nId_Persona = p_lider.nId_Persona

JOIN Usuarios u_lider

ON p_lider.nId_Persona = u_lider.nId_Persona

JOIN Colaboradores c_director

ON p.nId_Director = c_director.nId_Colaborador

JOIN Personas p_director

ON c_director.nId_Persona = p_director.nId_Persona

JOIN Usuarios u_director

ON p_director.nId_Persona = u_director.nId_Persona

WHERE u_lider.nId_Usuario = @nId_User

OR u_director.nId_Usuario = @nId_User;

  

--CREATE OBJECT #solicitudes

BEGIN

DECLARE @tabla_temp varchar(30);

SELECT

* INTO #requerimientos

FROM @Requerimientos_a_mostrar;

SET @tabla_temp = '#requerimientos'

END

  

SET @viewClause = N'select * from (select * from v_Listado_Requerimientos_refact where nId_Requerimiento in(select * from ' + @tabla_temp + '))a';

END

  

  

END

ELSE

BEGIN

set @viewClause = N'SELECT * FROM (SELECT * FROM v_Listado_Requerimientos_refact where nId_Proyecto ='+CONVERT(varchar, @Id_Proyecto)+') r'

END

  

--Set @whereClause

DECLARE @conditions_array TABLE(condition nvarchar(max))

DECLARE @condition AS nvarchar(max);

  

IF @fechaCreaIni IS NOT NULL AND @fechaCreaFin IS NOT NULL

BEGIN

EXEC sp_get_dates_condition 'dFecha_Creacion', @fechaCreaIni ,@fechaCreaFin, @condition OUTPUT

  

INSERT INTO @conditions_array VALUES(@condition)

END

  

IF @sNombreRequerimiento IS NOT NULL

BEGIN

IF @nTipoFilter = 1

BEGIN

-- Búsqueda por nombre de proyecto

EXEC sp_get_like_condition 'sNombre_Proyecto', @sNombreRequerimiento, @condition OUTPUT

INSERT INTO @conditions_array VALUES(@condition)

END

ELSE IF @nTipoFilter = 2

BEGIN

-- Búsqueda por nombre de requerimiento

EXEC sp_get_like_condition 'sNombre_Requerimiento', @sNombreRequerimiento, @condition OUTPUT

INSERT INTO @conditions_array VALUES(@condition)

END

END

  

IF @sFilterNotification IS NOT NULL

BEGIN

set @condition = 'nId_Requerimiento ='+@sFilterNotification+''

INSERT INTO @conditions_array VALUES(@condition)

END

  

DECLARE @count INT;

SELECT @count = COUNT(*) FROM @conditions_array;

if @count > 0

begin

set @whereClause = N'where ';

end

  

WHILE @count > 0

BEGIN

DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions_array);

IF @count = 1

BEGIN

SET @whereClause = concat (@whereClause,@condition_temp);

END

ELSE

BEGIN

SET @whereClause = concat (@whereClause,@condition_temp,' and ');

END

DELETE TOP (1) FROM @conditions_array

SELECT @count = COUNT(*) FROM @conditions_array;

END

--END Set @whereClause

  

--Set @orderClause

EXEC sp_get_order_clause @campo, @order , 'dFecha_Inicio' , @orderClause OUTPUT

--END Set @orderClause

  

--set @contador-registros

/*DECLARE @total INT

DECLARE @ParmDefinition NVARCHAR(500);

  

set @contador = concat('select @totalOut = count(*) from (',@viewClause,' ',@whereClause,')a');

SET @ParmDefinition = N'@totalOut int OUTPUT';

  

EXEC sp_executesql @contador,@ParmDefinition,@totalOut= @total OUTPUT*/

--end @contador

  

--Set @script

  

SET @script = concat ('select * from (',@viewClause,' ',@whereClause,')a ',@orderClause);

--END Set @script

--set @script = concat (@viewClause,' ',@whereClause,' ',@orderClause,' ',@paginationClause)

EXECUTE sp_executesql @script

print @script

  

END