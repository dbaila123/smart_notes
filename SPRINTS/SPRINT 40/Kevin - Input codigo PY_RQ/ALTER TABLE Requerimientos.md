```SLQ
ALTER TABLE Requerimientos ADD sPrefijo NVARCHAR(2) DEFAULT 'RQ' NOT NULL;  
  
  
update Requerimientos set sPrefijo = 'RQ'
```