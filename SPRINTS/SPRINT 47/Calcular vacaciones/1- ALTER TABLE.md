```SQL

-- COLABORADORES
ALTER TABLE Colaboradores
ADD dfec_ingreso_real DATETIME;

-- saldos_Vacaciones_Colaboradores
ALTER Table saldos_Vacaciones_Colaboradores
ADD nDias_Tomados_Vencidos_Laborales Decimal;

ALTER Table saldos_Vacaciones_Colaboradores
ADD nDias_Tomados_Vigente_Laborales Decimal;

ALTER Table saldos_Vacaciones_Colaboradores
ADD nDias_Tomados_Vencidos_Calendario Decimal;

ALTER Table saldos_Vacaciones_Colaboradores
ADD nDias_Tomados_Vigente_Calendario Decimal;
```