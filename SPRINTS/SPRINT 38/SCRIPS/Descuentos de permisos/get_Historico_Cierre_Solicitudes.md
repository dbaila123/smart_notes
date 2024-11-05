create or alter procedure get_Historico_Cierre_Solicitudes
as
begin
	select 
	hct.nId_Cierre,
	hct.dDatetime_Creacion,
	hct.dDatetime_Creacion Ultima_Fecha_Cierre,
	hct.Json_Detalles_Transacciones,
	hct.nUsuario_Creador
	FROM History_Closing_Transactions hct
end
go