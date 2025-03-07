```sql
INSERT INTO Categorias_Permisos (sNombre, sDescripcion,nEstado,nUsuario_Creador,dDatetime_Creador,nUsuario_Update,dDatetime_Update,nUsuario_Delete,dDatetime_Delete) values ('DESCUENTOS','DESCUENTOS DE CIERRES',1,172,'2025-02-28',NULL,NULL,NULL,NULL)

INSERT INTO Tipos_Solicitudes (sDescripcion,sComentario,nEstado,nJustificacion_Extra,nFiltro,nUsuario_Creador,dDatetime_Creador,nUsuario_Update,dDatetime_Update,nUsuario_Delete,dDatetime_Delete,nId_Categoria,nRangoMax,nRangoMin,nRecuperable,nArchivoObligatorio,nTipo_Frecuencia,nAfecta_Asistencia,nAplicar_Descuento,nCambio_Jornada,nListado_Fechas,nMostrar_Calendario,nTiempoAnticipacion,nDiasCalendario,nMostrar_Selector,nRangoMaxAdm,nRangoMinAdm,b_recuperable) 
VALUES ('VENCIMIENTO TOLERANCIA','Descuento de tolerancia despues del cierre de asistencia',1,0,1,172,GETDATE(),null,null,null,null,5,null,null,null,0,1,0,0,null,null,0,null,0,0,null,null,null);
```


