```SQL
CREATE OR ALTER PROCEDURE sp_Get_ProyectsFilter(
 @nId_Collab INT
)
AS
BEGIN
	DECLARE @CollaboratorsList VARCHAR(MAX);
	SET @CollaboratorsList = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @nId_Collab), 0, 0);

	DECLARE @Query NVARCHAR(MAX) = 'SELECT
	       p.nId_Proyecto,
	       CONCAT(p.sCodigo,  " ", p.sNombre) AS sNombre_Proyecto,
	       p.sEstado_Proyecto AS nEstado_Proyecto,
	       p.nId_Lider
	   FROM
	       Proyectos p
	   WHERE nId_Proyecto IN
	       (SELECT DISTINCT
	           nId_Proyecto
	       FROM Tareas t
	       WHERE nId_Colaborador IN (' + @CollaboratorsList + '))';

	EXEC sp_executesql @Query;
END
```