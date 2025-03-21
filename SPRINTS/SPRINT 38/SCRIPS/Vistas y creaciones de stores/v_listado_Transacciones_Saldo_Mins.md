

```sql
CREATE or ALTER VIEW v_listado_Transacciones_Saldo_Mins
AS
SELECT 
    tsm.nId_Colaborador,
    p.sPersona_Nombre AS SNombreColaborador,
    COUNT(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.nId_Transaccion END) AS Total_Solicitudes, -- Total de solicitudes solo para tipo 1
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalSuma, -- Sumar permisos tipo 1
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 9 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalResta, -- Sumar permisos tipo 2
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.dCantidad_Minutos ELSE 0 END) - 
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 9 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalDescontado, -- Total descontado
    CAST(MAX(tsm.dDatetime_Creacion) AS DATE) AS FechaUltimaTransaccion,
    c.sUrl_Foto

FROM 
    Transacciones_Saldo_Mins tsm
    JOIN Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador 
    JOIN Personas p ON p.nId_Persona = c.nId_Persona 
    JOIN Solicitudes s ON tsm.nId_Entidad = s.nId_Solictud
    join Tipos_Solicitudes ts on ts.nId_Tipo_Solicitud  = s.nId_Tipo_Solicitud 
where tsm.nId_Cierre IS NULL
		AND tsm.nId_Tipo_Entidad IN (1, 9)
		and ts.b_recuperable = 1 
GROUP BY 
    tsm.nId_Colaborador,
    p.sPersona_Nombre,
    c.sUrl_Foto;	
```

```sql
CREATE OR ALTER VIEW v_listado_Transacciones_Saldo_Mins
AS
SELECT 
    tsm.nId_Colaborador,
    p.sPersona_Nombre AS SNombreColaborador,
    COUNT(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.nId_Transaccion END) AS Total_Solicitudes, -- Total de solicitudes solo para tipo 1
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalSuma, -- Sumar permisos tipo 1
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 9 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalResta, -- Sumar permisos tipo 2
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.dCantidad_Minutos ELSE 0 END) - 
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 9 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalDescontado, -- Total descontado
    CAST(MAX(tsm.dDatetime_Creacion) AS DATE) AS FechaUltimaTransaccion,
    c.sUrl_Foto
FROM 
    Transacciones_Saldo_Mins tsm
    join Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador 
    join Personas p ON p.nId_Persona = c.nId_Persona 
    join Solicitudes s ON tsm.nId_Entidad = s.nId_Solictud and s.nEstado_Solicitud = 3
    join Tipos_Solicitudes ts on ts.nId_Tipo_Solicitud  = s.nId_Tipo_Solicitud 
where tsm.nId_Cierre IS NULL
		AND tsm.nId_Tipo_Entidad IN (1, 9)
		and ts.b_recuperable = 1 
GROUP BY 
    tsm.nId_Colaborador,
    p.sPersona_Nombre,
    c.sUrl_Foto;
    c.sUrl_Foto;		
```

```sql
create or alter VIEW v_listado_Transacciones_Saldo_Mins
AS
SELECT 
    tsm.nId_Colaborador,
    p.sPersona_Nombre AS SNombreColaborador,
    COUNT(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.nId_Transaccion END) AS Total_Solicitudes, -- Total de solicitudes solo para tipo 1
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalSuma, -- Sumar permisos tipo 1
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 9 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalResta, -- Sumar permisos tipo 2
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.dCantidad_Minutos ELSE 0 END) - 
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 9 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalDescontado, -- Total descontado
    CAST(MAX(tsm.dDatetime_Creacion) AS DATE) AS FechaUltimaTransaccion,
    c.sUrl_Foto
FROM 
    Transacciones_Saldo_Mins tsm
    join Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador 
    join Personas p ON p.nId_Persona = c.nId_Persona 
    join Solicitudes s ON tsm.nId_Entidad = s.nId_Solictud -- and s.nEstado_Solicitud = 3
	--
	join Configs cfgs on s.nEstado_Solicitud = cast (cfgs.sCodigo as int) and cfgs.sTabla = 'ESTADO_SOLICITUD' and cfgs.sCodigo_Externo = 'A'
    join Tipos_Solicitudes ts on ts.nId_Tipo_Solicitud  = s.nId_Tipo_Solicitud 
where tsm.nId_Cierre IS NULL
		AND tsm.nId_Tipo_Entidad IN (1, 9)
		and ts.b_recuperable = 1 
GROUP BY 
    tsm.nId_Colaborador,
    p.sPersona_Nombre,
    c.sUrl_Foto;
```

```sql
create or alter VIEW v_listado_Transacciones_Saldo_Mins
AS
SELECT
    tsm.nId_Colaborador,
    p.sPersona_Nombre AS SNombreColaborador,
    COUNT(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.nId_Transaccion END) AS Total_Solicitudes, -- Total de solicitudes solo para tipo 1
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalSuma, -- Sumar permisos tipo 1
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 9 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalResta, -- Sumar permisos tipo 2
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 1 THEN tsm.dCantidad_Minutos ELSE 0 END) - 
    SUM(CASE WHEN tsm.nId_Tipo_Entidad = 9 THEN tsm.dCantidad_Minutos ELSE 0 END) AS CantidadTotalDescontado, -- Total descontado
    CAST(MAX(tsm.dDatetime_Creacion) AS DATE) AS FechaUltimaTransaccion,
    c.sUrl_Foto,
	STRING_AGG(CAST(tsm.nId_Transaccion as varchar(max)), ',') sTransaccionesIds
FROM 
    Transacciones_Saldo_Mins tsm
    join Colaboradores c ON c.nId_Colaborador = tsm.nId_Colaborador 
    join Personas p ON p.nId_Persona = c.nId_Persona 
    join Solicitudes s ON tsm.nId_Entidad = s.nId_Solictud -- and s.nEstado_Solicitud = 3
	--
	join Configs cfgs on s.nEstado_Solicitud = cast (cfgs.sCodigo as int) and cfgs.sTabla = 'ESTADO_SOLICITUD' and cfgs.sCodigo_Externo = 'A'
    join Tipos_Solicitudes ts on ts.nId_Tipo_Solicitud  = s.nId_Tipo_Solicitud 
where tsm.nId_Cierre IS NULL
		AND tsm.nId_Tipo_Entidad IN (1, 9)
		and ts.b_recuperable = 1 
GROUP BY 
    tsm.nId_Colaborador,
    p.sPersona_Nombre,
    c.sUrl_Foto;
```