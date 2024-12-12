```sql
update Tipos_Solicitudes set nDiasCalendario = 0 where sDescripcion = 'CON GOCE DE HABER POR HORA'
go

update Tipos_Solicitudes set nDiasCalendario = 0 where sDescripcion = 'CON GOCE DE HABER POR DÍA'
go

UPDATE Tipos_Solicitudes set nRangoMax = 450 where sDescripcion = 'PERMISO POR HORAS'
go



declare @opcionId int, @rolId int
select @opcionId = nId_Opcion from Opciones where sSlug like '%COLABORADOR-D-INFOPERSONAL%'
select @rolId = nId_Rol from roles where sDescripcion = 'LÍDER'

if (@opcionId is not null and @rolId is not null)
begin
	MERGE Permisos_Opciones AS Destino
	-- USING Opciones AS Origen
	USING (SELECT @opcionId AS nId_Opcion, @rolId AS nId_Rol) AS Origen
	ON Destino.nId_opcion = Origen.nId_opcion and Destino.nid_rol = Origen.nid_rol
	WHEN MATCHED THEN
    -- Actualizar si el registro ya existe
    UPDATE SET 
        Destino.nEstado = 1
	WHEN NOT MATCHED BY TARGET THEN
    -- Insertar si no existe en la tabla destino
		INSERT (nId_Rol, nId_Opcion, nEstado, nUsuario_Creador, dDatetime_creador)
		VALUES (@rolId, @opcionId, 1, 78, GETDATE());
end
go

```