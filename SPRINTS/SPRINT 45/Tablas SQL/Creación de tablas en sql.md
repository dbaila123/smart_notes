```sql
 CREATE TABLE Evaluations (
    nId_Evaluation INT IDENTITY(1,1) PRIMARY KEY,
	sName NVARCHAR(500),
    sDescription NVARCHAR(MAX),
    dInit_Date DATE,
    dEnd_Date DATE,
	nUser_Creator INT,
    dDateTime_Creator DATETIME,
	nUser_Update INT,
    dDateTime_Update DATETIME,
	nUser_Delete INT,
    dDateTime_Delete DATETIME
);


CREATE TABLE Forms (
    nId_Form INT IDENTITY(1,1) PRIMARY KEY,
	sName NVARCHAR(500),
    sDescription NVARCHAR(MAX),
    nState INT,
    nId_Evaluation INT,
	nUser_Creator INT,
    dDateTime_Creator DATETIME,
	nUser_Update INT,
    dDateTime_Update DATETIME,
	nUser_Delete INT,
    dDateTime_Delete DATETIME,
	FOREIGN KEY (nId_Evaluation) REFERENCES Evaluations(nId_Evaluation)
);


CREATE TABLE Evaluators (
    nId_Evaluator INT,
	nId_Assessed INT,
    nState INT,
    nId_Form INT,
	FOREIGN KEY (nId_Evaluator) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Assessed) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Form) REFERENCES Forms(nId_Form)
);


CREATE TABLE Qualified_Evaluations (
    nId_Qualified_Evaluation INT IDENTITY(1,1) PRIMARY KEY,
	nId_Form INT,
	nId_Evaluator INT,
	nId_Assessed INT,
	nUser_Creator INT,
    dDateTime_Creator DATETIME,
	nUser_Update INT,
    dDateTime_Update DATETIME,
	nUser_Delete INT,
    dDateTime_Delete DATETIME,
	FOREIGN KEY (nId_Evaluator) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Assessed) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Form) REFERENCES Forms(nId_Form)
);

```