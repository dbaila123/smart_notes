```SQL
CREATE procedure [dbo].[sp_smart_insert_or_update_proyecto]

(

@nId_Usuario int,

@nId_Proyecto int = 0,

@sNombre_Sin_Prefijo varchar(max),

@sNombre varchar(max),

@sCodigo varchar(max),

@sPrefijo varchar(50),

@sPrefijo_Codigo varchar(10),

@sId_Tipo_Servicio varchar(max),

@nId_Lider int,

@nId_Director int,

@nId_Coordinador int,

@sDescripcion varchar(max),

@dFecha_Inicio date,

@dFecha_Fin date,

@nTipo_Acceso int = 0,

@action int

)

AS

BEGIN TRY

  

  

declare @code varchar(max)

declare @message varchar(max) --MENSAJE DE ERROR

declare @count int

declare @currentProject varchar(max)

  

declare @IdProyecto as int

declare @GeneratedCodigo varchar(max)

  

if @action = 1

begin

set @code = '200'

set @message = 'Proyecto registrado correctamente.'

  

set @count = (select

COUNT(*)

from Proyectos p

join Configs c

on c.sCodigo = p.sId_Tipo_Servicio

join Coordinadores_Proyectos cp

on cp.nId_Proyecto = p.nId_Proyecto

where p.sNombre = @sNombre

and cp.nId_Coordinador = @nId_Coordinador

and c.nEstado = 1

and c.sTabla = 'TIPO_SERVICIO')

  

if @count = 0

begin

if @sCodigo IS NULL

begin

-- Generamos el código como 'PROY-' seguido del ID del proyecto

set @GeneratedCodigo = CONCAT('PRY-', (SELECT ISNULL(MAX(nId_Proyecto), 0) + 1 FROM Proyectos))

end

else

begin

set @GeneratedCodigo = @sCodigo

end

insert into Proyectos

(

sNombre_Proyecto_Sin_Prefijo,

sNombre,

sCodigo,

sPrefijo,

sDescripcion,

sId_Tipo_Servicio,

nId_Lider,

nId_Director,

sEstado_Proyecto,

nEstado,

dFecha_Inicio,

dFecha_Fin,

nUsuario_Creador,

dDatetime_Creador,

nTipo_Acceso

)

values

(

@sNombre_Sin_Prefijo,

@sNombre,

@GeneratedCodigo,

@sPrefijo_Codigo,

@sDescripcion,

@sId_Tipo_Servicio,

@nId_Lider,

@nId_Director,

1,

1,

@dFecha_Inicio,

@dFecha_Fin,

@nId_Usuario,

DATEADD(hour,-5,getdate()),

@nTipo_Acceso

)

  

set @IdProyecto = (select SCOPE_IDENTITY())

  

insert into Coordinadores_Proyectos

(

nId_Coordinador,

nId_Proyecto,

nPrincipal,

nEstado,

nUsuario_Creador,

dDatetime_Creador

)

values

(

@nId_Coordinador,

@IdProyecto,

0,

1,

@nId_Usuario,

DATEADD(hour,-5,getdate())

)

  

select @code as 'Code', @message as 'Message', @IdProyecto 'IdProyecto'

  

end

  

else

begin

set @code = '400'

set @message = 'El '+@sNombre+' ya tiene un coordinador con el mismo nombre.'

  

select @code as 'Code', @message as 'Message', @IdProyecto 'IdProyecto'

end

end

  

if @action = 2

begin

set @code = '200'

set @message = 'Proyecto actualizado correctamente.'

  

set @currentProject = (select

p.sNombre

from Proyectos p

join Configs c

on c.sCodigo = p.sId_Tipo_Servicio

join Coordinadores_Proyectos cp

on cp.nId_Proyecto = p.nId_Proyecto

where p.sNombre = @sNombre

and cp.nId_Coordinador = @nId_Coordinador

and c.nEstado = 1

and c.sTabla = 'TIPO_SERVICIO')

  

declare @nId_Proyecto_out int

  

set @nId_Proyecto_out = (select

p.nId_Proyecto

from Proyectos p

join Configs c

on c.sCodigo = p.sId_Tipo_Servicio

join Coordinadores_Proyectos cp

on cp.nId_Proyecto = p.nId_Proyecto

where p.sNombre = @sNombre

and cp.nId_Coordinador = @nId_Coordinador

and c.nEstado = 1

and c.sTabla = 'TIPO_SERVICIO')

  

if @currentProject = @sNombre and @nId_Proyecto != @nId_Proyecto_out

begin

set @code = '400'

set @message = 'El '+@sNombre+' ya tiene un coordinador con el mismo nombre.'

/*

update Proyectos

set sDescripcion = @sDescripcion,

sId_Tipo_Servicio = @sId_Tipo_Servicio,

nId_Lider = @nId_Lider,

nId_Director = @nId_Director,

nUsuario_Update = @nId_Usuario,

dDatetime_Update = DATEADD(hour,-5,getdate())

where nId_Proyecto = @nId_Proyecto

print @code*/

select @code as 'Code', @message as 'Message', @nId_Proyecto 'IdProyecto'

end

else

begin

/*

set @count = (select

COUNT(*)

from Proyectos p

join Configs c

on c.sCodigo = p.sId_Tipo_Servicio

join Coordinadores_Proyectos cp

on cp.nId_Proyecto = p.nId_Proyecto

where p.sNombre = @sNombre_Sin_Prefijo

and cp.nId_Coordinador = @nId_Coordinador

and c.nEstado = 1

and c.sTabla = 'TIPO_SERVICIO')

  

if @count = 0

begin */

update Proyectos

set sNombre = @sNombre,

sNombre_Proyecto_Sin_Prefijo = @sNombre_Sin_Prefijo,

sPrefijo = @sPrefijo_Codigo,

sDescripcion = @sDescripcion,

sId_Tipo_Servicio = @sId_Tipo_Servicio,

nId_Lider = @nId_Lider,

nId_Director = @nId_Director,

dFecha_Inicio = @dFecha_Inicio,

dFecha_Fin = @dFecha_Fin,

nUsuario_Update = @nId_Usuario,

nTipo_Acceso = @nTipo_Acceso,

dDatetime_Update = DATEADD(hour,-5,getdate())

where nId_Proyecto = @nId_Proyecto

  

update Coordinadores_Proyectos

set nId_Coordinador = @nId_Coordinador,

nId_Proyecto = @nId_Proyecto,

nUsuario_Update = @nId_Usuario,

dDatetime_Update = DATEADD(hour,-5,getdate())

where nId_Proyecto = @nId_Proyecto

  

select @code as 'Code', @message as 'Message', @nId_Proyecto 'IdProyecto'

/*end

else

begin

set @code = '400'

set @message = 'El proyecto '+@sNombre_Sin_Prefijo+' ya tiene un coordinador con el mismo nombre....'

  

select @code as 'Code', @message as 'Message'

end*/

end

end

  

END TRY

BEGIN CATCH

SELECT ERROR_SEVERITY() AS 'Code',ERROR_MESSAGE() AS 'Message', @nId_Proyecto AS 'IdProyecto'

END CATCH
```