```SQL
	update PreferenciasColaborador  
	set sCards_Orden = REPLACE(sCards_Orden,']]', ',12]]')  
	where sCards_Orden not like '%12%'
```
