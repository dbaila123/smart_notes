```SQL
update PreferenciasColaborador  
set sCards_Orden = _REPLACE_(sCards_Orden,']]', ',12]]')  
where sCards_Orden not like '%12%'
```
