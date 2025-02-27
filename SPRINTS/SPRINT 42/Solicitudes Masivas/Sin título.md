```SQL
ALTER PROCEDURE sp_validate_overlap_requests_on_massive

@collaboratorId INT,

@startDate DATETIME,

@endDate DATETIME,

@startHour TIME,

@endHour TIME,

@affectAttendance BIT,

@frecuencyType INT,

@categoryId INT

AS

BEGIN

DECLARE @result NVARCHAR(200) = NULL

  

IF @frecuencyType = 1 -- Hour

BEGIN

-- Validate the input = Category != Mark + Task and nAfecta_Asistencia = 1

IF @categoryId != 4 AND @affectAttendance = 1

BEGIN

-- it cannot overlap to an HOUR RANGE of another hour request (Category != Mark + Task and nAfecta_Asistencia = 1)

SELECT @result = CONCAT('Dentro de un rango de ', ts.sDescripcion)

FROM Solicitudes s

JOIN Tipos_Solicitudes ts ON s.nId_Tipo_Solicitud = ts.nId_Tipo_Solicitud

WHERE s.nId_Colaborador = @collaboratorId AND

ts.nEstado = 1 AND

s.nEstado_Solicitud NOT IN (2,6,7) AND

ts.nId_Categoria = @categoryId AND

ts.nAfecta_Asistencia = 1 AND

-- check date

(

-- (s.dFecha_Inicio = @startDate AND s.dFecha_Fin = @endDate) OR

-- (s.dFecha_Inicio BETWEEN @startDate AND @endDate) OR

-- (s.dFecha_Fin BETWEEN @startDate AND @endDate)

  

(s.dFecha_Inicio BETWEEN @startDate AND @endDate)

OR

(s.dFecha_Fin BETWEEN @startDate AND @endDate)

OR

(@startDate BETWEEN s.dFecha_Inicio AND s.dFecha_Fin)

OR

(@endDate BETWEEN s.dFecha_Inicio AND s.dFecha_Fin)

) AND

-- check overlapping cases for hours

(

-- (s.dHora_Inicio BETWEEN @startHour AND @endHour) OR

-- (s.dHora_Fin BETWEEN @startHour AND @endHour)

  

(s.dHora_Inicio BETWEEN @startHour AND @endHour)

OR

(s.dHora_Fin BETWEEN @startHour AND @endHour)

OR

(@startHour BETWEEN s.dHora_Inicio AND s.dHora_Fin)

OR

(@endHour BETWEEN s.dHora_Inicio AND s.dHora_Fin)

)

  

-- it cannot overlap with the DATE RANGE of another request DATE RANGE: day or month

SELECT @result = CONCAT('Dentro de un rango de ', ts.sDescripcion)

FROM Solicitudes s

JOIN Tipos_Solicitudes ts ON s.nId_Tipo_Solicitud = ts.nId_Tipo_Solicitud

WHERE s.nId_Colaborador = @collaboratorId AND

ts.nEstado = 1 AND

s.nEstado_Solicitud NOT IN (2,6,7) AND

ts.nId_Categoria = @categoryId AND

ts.nAfecta_Asistencia = 1 AND

-- check date

-- (

-- (s.dFecha_Inicio = @startDate AND s.dFecha_Fin = @endDate) OR

-- (s.dFecha_Inicio BETWEEN @startDate AND @endDate) OR

-- (s.dFecha_Fin BETWEEN @startDate AND @endDate)

-- )

(

(s.dFecha_Inicio BETWEEN @startDate AND @endDate)

OR

(s.dFecha_Fin BETWEEN @startDate AND @endDate)

OR

(@startDate BETWEEN s.dFecha_Inicio AND s.dFecha_Fin)

OR

(@endDate BETWEEN s.dFecha_Inicio AND s.dFecha_Fin)

)

END

ELSE IF @categoryId = 4 -- Mark + Task

BEGIN

-- it cannot overlap to an HOUR RANGE of another hour request (Category = Mark + Task)

SELECT @result = CONCAT('Dentro de un rango de ', ts.sDescripcion)

FROM Solicitudes s

JOIN Tipos_Solicitudes ts ON s.nId_Tipo_Solicitud = ts.nId_Tipo_Solicitud

WHERE s.nId_Colaborador = @collaboratorId AND

ts.nEstado = 1 AND

ts.nTipo_Frecuencia = @frecuencyType AND

ts.nId_Categoria = @categoryId AND

ts.nId_Tipo_Solicitud != 4 AND -- extemporaneous mark

-- check date

-- (

-- (s.dFecha_Inicio = @startDate AND s.dFecha_Fin = @endDate) OR

-- (s.dFecha_Inicio BETWEEN @startDate AND @endDate) OR

-- (s.dFecha_Fin BETWEEN @startDate AND @endDate)

-- )

(

(s.dFecha_Inicio BETWEEN @startDate AND @endDate)

OR

(s.dFecha_Fin BETWEEN @startDate AND @endDate)

OR

(@startDate BETWEEN s.dFecha_Inicio AND s.dFecha_Fin)

OR

(@endDate BETWEEN s.dFecha_Inicio AND s.dFecha_Fin)

)

END

END

ELSE IF @frecuencyType = 2 OR @frecuencyType = 3 -- Day or Month

BEGIN

-- Validate the input: nAfecta_Asistencia = 1

IF @affectAttendance = 1

BEGIN

-- it cannot overlap DATE to another request DATE RANGE: hour, day or month (nAfecta_Asistencia = 1)

SELECT @result = CONCAT('Dentro de un rango de ', ts.sDescripcion)

FROM Solicitudes s

JOIN Tipos_Solicitudes ts ON s.nId_Tipo_Solicitud = ts.nId_Tipo_Solicitud

WHERE s.nId_Colaborador = @collaboratorId AND

ts.nEstado = 1 AND

s.nEstado_Solicitud NOT IN (2,6,7) AND

--ts.nTipo_Frecuencia = @frecuencyType AND

ts.nAfecta_Asistencia = 1 AND

-- check date

-- (

-- (s.dFecha_Inicio = @startDate AND s.dFecha_Fin = @endDate) OR

-- (s.dFecha_Inicio BETWEEN @startDate AND @endDate) OR

-- (s.dFecha_Fin BETWEEN @startDate AND @endDate)

-- )

(

(s.dFecha_Inicio BETWEEN @startDate AND @endDate)

OR

(s.dFecha_Fin BETWEEN @startDate AND @endDate)

OR

(@startDate BETWEEN s.dFecha_Inicio AND s.dFecha_Fin)

OR

(@endDate BETWEEN s.dFecha_Inicio AND s.dFecha_Fin)

)

END

END

  

SELECT @result

END```