CREATE OR ALTER PROC get_footer_closure_history
    @PageNumber INT,
    @PageSize INT
AS
BEGIN
    SELECT 
        hct.nId_Cierre AS nId_Cierre,
        hct.dDatetime_Creacion AS dDatetime_Creacion,
        (SELECT MAX(dDatetime_Creacion) FROM History_Closing_Transactions) AS Ultima_Fecha_Cierre,
        hct.Json_Detalles_Transacciones AS Json_Detalles_Transacciones,
        hct.nUsuario_Creador AS nUsuario_Creador,
        CONCAT_WS(' ', p.sPrimer_Nombre, p.sApe_Paterno) AS sUsuarioCierre,
        COUNT(*) OVER() AS nTotal
    FROM History_Closing_Transactions hct
    INNER JOIN Usuarios u ON hct.nUsuario_Creador = u.nId_Usuario
    INNER JOIN Personas p ON p.nId_Persona = u.nId_Persona
    ORDER BY hct.nId_Cierre DESC
    OFFSET (@PageNumber - 1) * @PageSize ROWS
    FETCH NEXT @PageSize ROWS ONLY;
END;
GO