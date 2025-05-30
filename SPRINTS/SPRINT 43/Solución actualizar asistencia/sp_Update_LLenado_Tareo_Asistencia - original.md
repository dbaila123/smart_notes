```sql
CREATE   PROCEDURE [dbo].[sp_Update_LLenado_Tareo_Asistencia]  
(  
 @dFecha_Asistencia date,  
 @IdColaboradorIndividual int = 0/*Si no es null para regularizar solo aun colaborador y una sola fecha*/  
)  
AS  
BEGIN  
  
 DECLARE @dDia_de_la_semana int = (SELECT DATEPART(DW, @dFecha_Asistencia)-1);  
 DECLARE @nId_Colaboradores TABLE(nId_Colaborador int, nId_Horario int, nTotalJornada int)  
 
 IF @IdColaboradorIndividual > 0  
  BEGIN  
   INSERT INTO @nId_Colaboradores (nId_Colaborador, nId_Horario, nTotalJornada)  
   SELECT asi.nId_Colaborador, asi.nId_Horario, ht.nTotalJornada  
   FROM Asistencias asi  
   left join Horario_Total_Detalle ht on ht.nId_Horario = asi.nId_Horario and ht.nDia = @dDia_de_la_semana  
   WHERE dFecha_Asistencia = @dFecha_Asistencia AND nId_Colaborador =  @IdColaboradorIndividual  
   group by asi.nId_Colaborador, asi.nId_Horario, ht.nTotalJornada  
  END  
 ELSE  
  BEGIN  
   INSERT INTO @nId_Colaboradores (nId_Colaborador, nId_Horario, nTotalJornada)  
   SELECT DISTINCT asi.nId_Colaborador, asi.nId_Horario, ht.nTotalJornada  
   FROM Asistencias asi  
   inner join Horario_Total_Detalle ht on ht.nId_Horario = asi.nId_Horario and ht.nDia = @dDia_de_la_semana  
   WHERE dFecha_Asistencia = @dFecha_Asistencia  
   group by asi.nId_Colaborador, asi.nId_Horario, ht.nTotalJornada  
   --AND nId_Colaborador not in (1,2)  
  END  
  
 DECLARE @nId_Colaborador int  
 DECLARE @nId_Horario int  
 DECLARE @nTotalJornada int  
  
 -- Recorrer los nId_Colaboradores uno por uno  
 WHILE EXISTS (SELECT 1 FROM @nId_Colaboradores)  
 BEGIN  
  -- Obtener el siguiente nId_Colaborador  
  SELECT TOP 1  
  @nId_Colaborador = nId_Colaborador,  
  @nId_Horario = nId_Horario,  
  @nTotalJornada = nTotalJornada  
  FROM @nId_Colaboradores  
  
  
 DECLARE @TieneMarca int  
 DECLARE @Requerido int  
 DECLARE @Llenado int  
 DECLARE @nRequerido_No_Regular int  
 DECLARE @nLlenadoNo_Regular int  
 DECLARE @EstadoLlenadoTareo nvarchar(max)  
 DECLARE @EstadoHorarioTareo nvarchar(max)  
 DECLARE @MinutosSolicitud int  
 DECLARE @HoraEntrada time  
 DECLARE @HoraSalida time  
 DECLARE @Tardanza_colaborador int
  
 -- Obtener la tardanza del colaborador (asegurando que no sea NULL)
 SET @Tardanza_colaborador = ISNULL((SELECT TOP(1) dTiempo_Tardanza 
                               FROM Asistencias 
                               WHERE dFecha_Asistencia = @dFecha_Asistencia 
                               AND nId_Colaborador = @nId_Colaborador), 0)
  
  SET @HoraEntrada = (SELECT Entrada FROM(SELECT  
  CASE  
   WHEN (  
    SELECT COUNT(*)  
      FROM v_horario_dias_total_horas  
      WHERE nId_Horario = @nId_Horario  
      AND nDia = @dDia_de_la_semana  
     ) > 0  
   THEN (  
     SELECT Entrada  
     FROM v_horario_dias_total_horas  
     WHERE nId_Horario = @nId_Horario  
     AND nDia = @dDia_de_la_semana  
    )  
  ELSE '00:00:00'  
  END Entrada  
  )TEMP_HORA_ENTRADA )  
  
  
  
  SET @HoraSalida = (SELECT Salida FROM(SELECT  
  CASE  
   WHEN (  
    SELECT COUNT(*)  
      FROM v_horario_dias_total_horas  
      WHERE nId_Horario = @nId_Horario  
      AND nDia = @dDia_de_la_semana  
     ) > 0  
   THEN (  
     SELECT Salida  
     FROM v_horario_dias_total_horas  
     WHERE nId_Horario = @nId_Horario  
     AND nDia = @dDia_de_la_semana  
    )  
  ELSE '00:00:00'  
  END Salida  
  )TEMP_HORA_SALIDA)  
  
  SET @TieneMarca = ( SELECT COUNT(*)  
       FROM Marcas m  
       WHERE nId_Colaborador = @nId_Colaborador  
       AND dFecha_Jornada = @dFecha_Asistencia  
       AND nEstado IN (0,1))  
  
  -- Cálculo de @Requerido con manejo de NULL
  SET @Requerido = ISNULL(  
    (SELECT nRequerido FROM (SELECT  
         CASE  
          WHEN (SELECT COUNT(*) FROM Feriados f WHERE f.dFecha = @dFecha_Asistencia) > 0  
          THEN 0  
  
          WHEN (  
      SELECT COUNT(*) from v_horario_dias_total_horas s2  
           -- JOIN Horarios_Colaboradores hc2 on hc2.nId_Colaborador = @nId_Colaborador  
            WHERE s2.nId_Horario = @nId_Horario  and s2.nDia = @dDia_de_la_semana  
     ) > 0  
     THEN  
      (  
      SELECT  
  (CASE  
   WHEN vhdt.nTotal_Minutos - ISNULL(nMinutos_Permiso.nMinutos,0) IS NULL OR vhdt.nTotal_Minutos - ISNULL(nMinutos_Permiso.nMinutos,0) < 0  
   THEN 0  
   ELSE vhdt.nTotal_Minutos - ISNULL(nMinutos_Permiso.nMinutos,0)  
  END) as MinutosLlenar  
 FROM v_horario_dias_total_horas vhdt  
 LEFT JOIN ( SELECT  
    s.nId_Colaborador,  
       SUM(CASE  
               WHEN ts.nTipo_Frecuencia IN (2, 3)  
               THEN COALESCE(vhdth.nTotal_Minutos, 0)  
               WHEN ts.nTipo_Frecuencia = 1  
               THEN COALESCE(s.nCantidad, 0)  
               END) AS nMinutos  
       from Solicitudes s  
       join Tipos_Solicitudes ts on ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud and ts.nTipo_Frecuencia != 0  
       JOIN v_horario_dias_total_horas vhdth on vhdth.nId_Horario = @nId_Horario and vhdth.nDia = @dDia_de_la_semana  
       WHERE @dFecha_Asistencia BETWEEN s.dFecha_Inicio and s.dFecha_Fin  
        AND ts.nId_Tipo_Solicitud not in (20,21,10)  
        AND ts.nId_Categoria not in (4,5)  
        AND s.nEstado_Solicitud in (1,3)  
        AND s.nId_Colaborador = @nId_Colaborador
        GROUP BY s.nId_Colaborador  
 ) nMinutos_Permiso ON nMinutos_Permiso.nId_Colaborador = @nId_Colaborador  
 WHERE vhdt.nId_Horario = @nId_Horario and vhdt.nDia = @dDia_de_la_semana  
      )  
    ELSE  
    0  
         END nRequerido ) TEMP_REQUERIDO ), 0)
  
  -- Asegurarse de que @Requerido no sea NULL después de restar la tardanza
  SET @Requerido = ISNULL(@Requerido - @Tardanza_colaborador, 0)
  
  -- Restringir valor negativo de @Requerido
  IF @Requerido < 0
     SET @Requerido = 0
  
  SET @Llenado = (SELECT ISNULL(SUM(nMinutos), 0)  
       FROM Tareas t  
       WHERE nId_Colaborador = @nId_Colaborador  
       AND dFecha_Registro = @dFecha_Asistencia  
       AND nEstado_Tarea NOT IN (6)  
       AND sTipo_Hora = '1')  
  
  SET @nLlenadoNo_Regular = (SELECT ISNULL(SUM(nMinutos), 0)  
       FROM Tareas t  
       WHERE nId_Colaborador = @nId_Colaborador  
       AND dFecha_Registro = @dFecha_Asistencia  
       AND nEstado_Tarea NOT IN (6)  
       AND sTipo_Hora != '1')  
  
  SET @nRequerido_No_Regular = 1440 - @Requerido  
  
  SET @EstadoLlenadoTareo = (SELECT sEstadoLLenadoTareo FROM (SELECT  
         CASE  
          WHEN (  
           SELECT COUNT(*) from v_horario_dias_total_horas s2  
           -- JOIN Horarios_Colaboradores hc2 on hc2.nId_Colaborador = @nId_Colaborador  
            WHERE s2.nId_Horario = @nId_Horario and s2.nDia = @dDia_de_la_semana  
            ) = 0  
          THEN 'COMPLETO'  
  
          WHEN (SELECT COUNT(*) FROM Feriados f WHERE f.dFecha = @dFecha_Asistencia) > 0  
          THEN 'COMPLETO'  
  
          WHEN (  
            ( -- david  
             SELECT COUNT(*)  
             FROM Solicitudes s  
             JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud  
             WHERE s.nEstado_Solicitud in (1,3)  
             AND s.nId_Tipo_Solicitud NOT IN (21)  
             AND @dFecha_Asistencia BETWEEN s.dFecha_Inicio AND s.dFecha_Fin  
             AND ts.nAfecta_Asistencia = 1  
             AND (ts.nTipo_Frecuencia = 2 or ts.nTipo_Frecuencia = 3)  
             AND s.nId_Colaborador = @nId_Colaborador  
             )> 0  
             ) -- david  
             or ((  
             SELECT SUM(nCantidad)  
             FROM Solicitudes s  
             JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud  
             WHERE s.nEstado_Solicitud in (1,3)  
             AND s.nId_Tipo_Solicitud NOT IN (21)  
             AND @dFecha_Asistencia BETWEEN s.dFecha_Inicio AND s.dFecha_Fin  
             AND ts.nAfecta_Asistencia = 1  
             AND s.nId_Colaborador = @nId_Colaborador  
             )>= @nTotalJornada)  
          THEN 'COMPLETO'  
  
          WHEN @Llenado = 0  
          THEN 'FALTA'  
  
          WHEN @Requerido <= @Llenado  
          THEN 'COMPLETO'  
  
          WHEN (@Requerido - @Llenado) > 0  
          THEN 'INCOMPLETO'  
  
         END sEstadoLLenadoTareo ) TEMP_ESTADO_LLENADO_TAREO)  
  
  SET @EstadoHorarioTareo = (SELECT sEstadoTareoHorario FROM (SELECT  
          CASE  
           WHEN (SELECT COUNT(*) FROM Feriados f WHERE f.dFecha = @dFecha_Asistencia) > 0  
           THEN 'FERIADO'  
  
           -- cambio david  
           WHEN (  
             (  
               SELECT COUNT(*)  
               FROM Solicitudes s  
               JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud                 
			   WHERE s.nEstado_Solicitud in (1,3)  
               AND s.nId_Tipo_Solicitud NOT IN (21)  
               AND @dFecha_Asistencia BETWEEN s.dFecha_Inicio AND s.dFecha_Fin  
               AND ts.nAfecta_Asistencia = 1  
               AND (ts.nTipo_Frecuencia = 2 or ts.nTipo_Frecuencia = 3)  
               AND s.nId_Colaborador = @nId_Colaborador  
               )> 0  
             ) or (  
             (  
              SELECT SUM (nCantidad)  
              FROM Solicitudes s  
              JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud  
              WHERE s.nEstado_Solicitud in (1,3)  
              AND s.nId_Tipo_Solicitud NOT IN (21)  
              AND @dFecha_Asistencia BETWEEN s.dFecha_Inicio AND s.dFecha_Fin  
              AND ts.nAfecta_Asistencia = 1  
              AND s.nId_Colaborador = @nId_Colaborador  
              ) >= @nTotalJornada  
             )  
           THEN 'SOLICITUD_POR_DIAS'  
  
           WHEN (  
             SELECT COUNT(s.nId_Solictud)  
             FROM Solicitudes s  
             JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud  
             WHERE s.nEstado_Solicitud in (1,3)  
             AND @dFecha_Asistencia BETWEEN s.dFecha_Inicio AND s.dFecha_Fin  
             AND ts.nAfecta_Asistencia = 1  
             AND ts.nTipo_Frecuencia = 1  
             AND s.nId_Colaborador = @nId_Colaborador  
             ) > 0  
           THEN 'SOLICITUD_POR_HORAS'  
  
           WHEN (SELECT COUNT(*) from v_horario_dias_total_horas s2  
             --JOIN Horarios_Colaboradores hc2 on hc2.nId_Colaborador = @nId_Colaborador  
             WHERE s2.nId_Horario = @nId_Horario  and s2.nDia = @dDia_de_la_semana) > 0  
  
           THEN 'HORARIO_REGULAR'  
           ELSE  
            'FUERA_HORARIO_REGULAR'  
          END sEstadoTareoHorario ) TEMP_ESTADO_TAREO_HORARIO)  
  
  
  set @MinutosSolicitud = ISNULL((SELECT nMinutosPermiso FROM (SELECT  
          CASE  
           WHEN (SELECT COUNT(*) from v_horario_dias_total_horas s2  
            --JOIN Horarios_Colaboradores hc2 on hc2.nId_Colaborador = @nId_Colaborador  
            WHERE s2.nId_Horario = @nId_Horario and s2.nDia = @dDia_de_la_semana ) =0  
  
           THEN 0  
  
           WHEN (  
             SELECT COUNT(*)  
             FROM Solicitudes s  
             JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud  
             WHERE s.nEstado_Solicitud in (1,3)  
             AND s.nId_Tipo_Solicitud NOT IN (21)  
             AND @dFecha_Asistencia BETWEEN s.dFecha_Inicio AND s.dFecha_Fin  
             AND ts.nAfecta_Asistencia = 1  
             AND (ts.nTipo_Frecuencia = 2 or ts.nTipo_Frecuencia = 3)  
             AND s.nId_Colaborador = @nId_Colaborador  
             )> 0  
           THEN (  
            SELECT nTotal_Minutos from v_horario_dias_total_horas s2  
         -- JOIN Horarios_Colaboradores hc2 on hc2.nId_Colaborador = @nId_Colaborador  
          WHERE s2.nId_Horario = @nId_Horario  and s2.nDia = @dDia_de_la_semana  
          )  
  
           WHEN (  
             SELECT COUNT(*)  
             FROM Solicitudes s  
             JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud  
             WHERE s.nEstado_Solicitud in (1,3)  
             AND @dFecha_Asistencia BETWEEN s.dFecha_Inicio AND s.dFecha_Fin  
             AND ts.nAfecta_Asistencia = 1  
             AND ts.nTipo_Frecuencia = 1  
             AND s.nId_Colaborador = @nId_Colaborador  
             ) > 0  
           THEN (  
             SELECT SUM(s.nCantidad)  
             FROM Solicitudes s  
             JOIN Tipos_Solicitudes ts ON ts.nId_Tipo_Solicitud = s.nId_Tipo_Solicitud  
             WHERE s.nEstado_Solicitud in (1,3)  
             AND @dFecha_Asistencia BETWEEN s.dFecha_Inicio AND s.dFecha_Fin  
             AND ts.nAfecta_Asistencia = 1  
             AND ts.nTipo_Frecuencia = 1  
             AND s.nId_Colaborador = @nId_Colaborador  
             )  
           ELSE  
            0  
          END nMinutosPermiso ) TEMP_MINUTOS_SOLICITUD), 0)  
  
  UPDATE Asistencias  
  SET nLlenado = @Llenado, 
      nRequerido = @Requerido, 
      nLlenadoNo_Regular = @nLlenadoNo_Regular, 
      nRequerido_No_Regular = @nRequerido_No_Regular, 
      nTiene_Marca = @TieneMarca, 
      sEstadoLLenadoTareo = @EstadoLlenadoTareo,  
      sEstadoTareoHorario = @EstadoHorarioTareo, 
      nMinutos_Permiso = @MinutosSolicitud,  
      tHoraEntrada = @HoraEntrada, 
      tHoraSalida = @HoraSalida  
  WHERE nId_Colaborador = @nId_Colaborador  
  AND dFecha_Asistencia = @dFecha_Asistencia  
  AND nEstado_Asistencia != 11  
  
  -- Eliminar el nId_Colaborador procesado de la variable de tabla  
  DELETE FROM @nId_Colaboradores WHERE nId_Colaborador = @nId_Colaborador  
 END  
END
```
