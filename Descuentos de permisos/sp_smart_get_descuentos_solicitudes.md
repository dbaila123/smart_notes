```sql
alter procedure sp_smart_get_descuentos_solicitudes
(
	@nId INT,
	@numFilter int
)
 AS
 BEGIN
 		SELECT 
 	s.nId_Solictud,
 	c.nId_Colaborador,
 	p.sPersona_Nombre AS SNombreColaborador,
 	tsm.nId_Sub_Tipo_Entidad,
 	tsm.nId_Tipo_Entidad,
 	tsm.nTipo_Transaccion,
    s.nId_Tipo_Solicitud,
    
    s.dDatetime_Creador,
    s.nUsuario_Creador,
        SUM(s.nCantidad) AS TotalCantidad,  -- Agregación para sumar cantidades
    CONVERT(VARCHAR(8), MIN(s.dHora_Inicio), 108) AS dHora_Inicio,  -- Mínima hora de inicio
        CONVERT(VARCHAR(8), MAX(s.dHora_Fin), 108) AS dHora_Fin, 
    s.sMotivo,
    
    s.nId_Supervisor,
    s.nId_Tipo_Motivo,
    s.sObservacion_Lider,
    s.sAprobadores,
    s.dFecha_Solicitud,
 	ts.nId_Categoria,
 	CONCAT(p.sPrimer_Nombre, ' ', p.sApe_Paterno) AS sNombre_Lider,
 	
  	cs.nId_Supervisor AS nId_Lider,
  	s.nEstado_Solicitud,
  	se.nId_Sede nSede,
  	se.sDescripcion sNombre_Sede,
  	ts.nTipo_Frecuencia,
  	
  	tb_conf_modalidad.sDescripcion sNombre_Modalidad,
  	tb_conf_modalidad.sIcono sIcono_Tipo_Modalidad,
  	tm.sDescripcion sNombre_Tipo_Motivo,
  	ts.sDescripcion sNombre_Tipo_Solicitud,
  	peco.sUrl_Foto
    FROM Transacciones_Saldo_Mins tsm
    JOIN Solicitudes s ON tsm.nId_Entidad = s.nId_Solictud
    JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador 
    JOIN Personas p ON p.nId_Persona = c.nId_Persona 
    JOIN Tipos_Solicitudes ts on ts.nId_Tipo_Solicitud  = s.nId_Tipo_Solicitud 
    JOIN vw_user_person_collab peco ON s.nId_Colaborador = peco.nId_Colaborador
    JOIN colaboradores_supervisor cs ON cs.nId_Colaborador = s.nId_Colaborador
    JOIN Sedes_Colaboradores sc ON peco.nId_Colaborador = sc.nId_Colaborador
    JOIN Sedes se ON se.nId_Sede = sc.nId_Sede
    JOIN Configs tb_conf_modalidad ON CAST(peco.nTipo_Modalidad as VARCHAR(10)) = tb_conf_modalidad.sCodigo AND tb_conf_modalidad.sTabla = 'TIPO_MODALIDAD'
    JOIN Tipos_Motivos tm ON tm.nId_Tipo_Motivo = s.nId_Tipo_Motivo AND cs.bPrincipal = 1
    WHERE tsm.nId_Colaborador = @numFilter and 
    ts.b_recuperable = 1 and
     nId_Cierre IS NULL
     GROUP BY 
        s.nId_Solictud,
        c.nId_Colaborador,
        p.sPersona_Nombre,
        tsm.nId_Sub_Tipo_Entidad,
        tsm.nId_Tipo_Entidad,
        tsm.nTipo_Transaccion,
        s.nId_Tipo_Solicitud,
        s.dDatetime_Creador,
        s.nUsuario_Creador,
        s.sMotivo,
        s.nId_Supervisor,
        s.nId_Tipo_Motivo,
        s.sObservacion_Lider,
        s.sAprobadores,
        s.dFecha_Solicitud,
        ts.nId_Categoria,
        CONCAT(p.sPrimer_Nombre, ' ', p.sApe_Paterno),
        cs.nId_Supervisor,
        s.nEstado_Solicitud,
        se.nId_Sede,
        se.sDescripcion,
        ts.nTipo_Frecuencia,
        tb_conf_modalidad.sDescripcion,
        tb_conf_modalidad.sIcono,
        tm.sDescripcion,
        ts.sDescripcion,
        peco.sUrl_Foto;
END

```