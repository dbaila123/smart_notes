```sql
DECLARE @nId_colaborador INT
DECLARE @nId_Usuario INT
DECLARE @nId_Tareo INT
DECLARE @nId_Solicitud INT
DECLARE @nId_Asistencia INT
DECLARE @nId_Director INT
DECLARE @dFecha_Registro date
DECLARE @nId_Típo INT
DECLARE @nId_Proyecto INT
DECLARE @nId_Requerimiento INT
DECLARE @nId_Supervisor INT
DECLARE @nId_Lider_Proyecto NVARCHAR(MAX)
DECLARE @nId_Lider_Requerimiento NVARCHAR(MAX)
DECLARE @nId_Director_Proyecto NVARCHAR(MAX)
DECLARE @nMinutos INT
DECLARE @sComentario NVARCHAR(MAX)
DECLARE @nHora INT
DECLARE @nEstado_Tarea INT
DECLARE @nEstado_Solicitud INT
DECLARE @nCategoria_Tarea INT
DECLARE @nTipo_Actividad  INT
DECLARE @nTipo_Solicitud INT
DECLARE @sHora_Inicio NVARCHAR(MAX)
DECLARE @sHora_Fin NVARCHAR(MAX)
DECLARE @dFecha_Inicio date
DECLARE @dFecha_Fin date
DECLARE @nCodigo_Historico INT

BEGIN TRANSACTION;

BEGIN TRY
	SET @nId_Usuario = 253
	SET @nId_Colaborador = 
		(
			SELECT TOP 1 c.nId_Colaborador
				FROM Usuarios u
				INNER JOIN Colaboradores c ON u.nId_Persona = c.nId_Persona
			WHERE u.nId_Usuario = 253
		)
	SET @dFecha_Registro = '2024-12-15'
	SET @nId_Proyecto = (SELECT nId_Proyecto FROM proyectos where sNombre = 'Reaseguros VT')
	SET @nMinutos = 360
	SET @sComentario = 'Homologación de script y procedimientos almacenados para pase a producción de REASEGUROS'
	SET @sHora_Inicio ='14:00:00'
	SET @sHora_Fin ='18:00:00'
	SET @dFecha_Inicio ='2024-12-15'
	SET @dFecha_Fin ='2024-12-15'
	SET @nHora = (select sCodigo from configs where sTabla = 'TIPO_HORA' and sDescripcion = 'EXTRAS')
	SET @nEstado_Tarea = (select sCodigo from configs where sTabla = 'ESTADO_TAREA' and sDescripcion = 'APROBADO')
	SET @nCategoria_Tarea = (select sCodigo from configs where sTabla = 'CATEGORIA_TAREA' and sDescripcion = 'PASE A PRODUCCIÓN')
	SET @nTipo_Actividad = (select sCodigo from configs where sTabla = 'TIPO_ACTIVIDAD' and sDescripcion = 'APOYO')
	SET @nTipo_Solicitud = (SELECT nId_Tipo_Solicitud FROM Tipos_Solicitudes where sDescripcion = 'HORAS EXTRAS')
	SET @nEstado_Solicitud = (select sCodigo from configs where sTabla = 'ESTADO_SOLICITUD' and sDescripcion = 'APROBADO')
	SET @nCodigo_Historico = (select sCodigo from configs where sTabla = 'TIPO_HISTORICO' and sDescripcion = 'APROBACIÓN')
	SET @nId_Requerimiento = (SELECT nId_Requerimiento FROM Requerimientos where nId_Proyecto = @nId_Proyecto and sNombre_SinPrefijo = 'PASE A PRODUCCION')
	
	SET @nId_Asistencia = (SELECT nId_Asistencia FROM asistencias where nId_Colaborador = @nId_Colaborador and dFecha_Asistencia = @dFecha_Registro)
	SET @nId_Director_Proyecto = (SELECT nId_Director FROM Proyectos where nId_Proyecto = @nId_Proyecto)
	SET @nId_Lider_Proyecto = (SELECT nId_Lider FROM Proyectos WHERE nId_Proyecto = @nId_Proyecto)
	SET @nId_Lider_Requerimiento = (SELECT nId_Lider FROM Requerimientos WHERE nId_Proyecto = @nId_Proyecto)
	SET @nId_Supervisor = (SELECT nId_Supervisor FROM colaboradores_supervisor WHERE nId_Colaborador = @nId_colaborador)
	
	INSERT INTO tareas (
	    nId_Proyecto,
	    dFecha_Registro,
	    sId_Categoria,
	    sDetalle,
	    nMinutos,
	    sAnotaciones,
	    nEstado,
	    nUsuario_Creador,
	    dDatetime_Creador,
	    nUsuario_Update,
	    dDatetime_Update,
	    nUsuario_Delete,
	    dDatetime_Delete,
	    nEstado_Tarea,
	    nId_Colaborador,
	    sTipo_Actividad,
	    sTipo_Hora,
	    nId_Requerimiento,
	    nIdDirector,
	    nId_Asitencia,
	    nId_Director_Pry,
	    nId_Lider,
	    nId_Lider_Pry,
	    nId_Lider_RQ,
	    nEstado_Pago,
	    nId_Cierre
	) VALUES (
	    @nId_Proyecto,
	    @dFecha_Registro,
	    @nCategoria_Tarea,
	    @sComentario,
	    @nMinutos,
	    NULL,
	    1,
	    @nId_Usuario,
	    @dFecha_Registro,
	    NULL,
	    @dFecha_Registro,
	    NULL,
	    NULL,
	    @nEstado_Tarea,
	    @nId_colaborador,
	    @nTipo_Actividad,
	    @nHora,
	    @nId_Requerimiento,
	    @nId_Director_Proyecto,
	    @nId_Asistencia,
	    @nId_Director_Proyecto,
	    @nId_Lider_Proyecto,
	    @nId_Lider_Proyecto,
	    @nId_Lider_Requerimiento,
	    NULL,
	    NULL
	);
	
	SET @nId_Tareo = SCOPE_IDENTITY();
	PRINT(@nId_Tareo)
	
	INSERT INTO solicitudes (
	    nId_Tipo_Solicitud,
	    nId_Colaborador,
	    nId_Tipo_Motivo,
	    nEstado_Solicitud,
	    nCantidad,
	    dFecha_Solicitud,
	    dHora_Inicio,
	    dHora_Fin,
	    dFecha_Inicio,
	    dFecha_Fin,
	    sMotivo,
	    sObservacion_Lider,
	    nEstado,
	    nUsuario_Creador,
	    dDatetime_Creador,
	    nUsuario_Update,
	    dDatetime_Update,
	    nUsuario_Delete,
	    dDatetime_Delete,
	    sObservacion_Director,
	    nId_Tarea,
	    nId_Proyecto,
	    nId_Team,
	    nRecuperable,
	    sAprobadores,
	    nId_Supervisor
	) VALUES (
	    @nTipo_Solicitud,
	    @nId_colaborador,
	    17,
	    @nEstado_Solicitud,
	    @nMinutos,
	    @dFecha_Registro,
	    @sHora_Inicio,
	    @sHora_Fin,
	    @dFecha_Inicio,
	    @dFecha_Fin,
	    @sComentario,
	    NULL,
	    1,
	    @nId_Usuario,
	    @dFecha_Registro,
	    NULL,
	    @dFecha_Registro,
	    NULL,
	    NULL,
	    NULL,
	    @nId_Tareo,
	    @nId_Proyecto,
	    NULL,
	    NULL,
	    '[]',
	    @nId_Supervisor
	);
	
	SET @nId_Solicitud = SCOPE_IDENTITY();
	PRINT (@nId_Solicitud)
	
	INSERT INTO Tareas_Historicos
	(
		nId_Tarea,
		sJson_Tarea_Old,
		nEstado,
		nUsuario_Creador,
		dDatetime_Creador,
		nUsuario_Update,
		dDatetime_Update,
		nUsuario_Delete,
		dDatetime_Delete,
		sCambios,
		nTipo_Historico,
		nId_Solicitud
	) VALUES (
		@nId_Tareo,
		null,
		1,
		@nId_Usuario,
		@dFecha_Registro,
		NULL,
		NULL,
		NULL,
		NULL,
		'<strong>Tipo de actividad registrada por service desk</strong>',
		@nCodigo_Historico,
		@nId_Solicitud
	)
END TRY
BEGIN CATCH
	ROLLBACK TRANSACTION;
END CATCH
```
