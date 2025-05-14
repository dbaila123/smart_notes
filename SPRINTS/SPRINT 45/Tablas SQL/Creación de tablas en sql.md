```sql
 CREATE TABLE Evaluations (
    nId_Evaluation INT IDENTITY(1,1) PRIMARY KEY,
	sName NVARCHAR(500) NOT NULL,
    nState INT NOT NULL,
    sDescription NVARCHAR(MAX),
    dInit_Date DATE NOT NULL,
    dEnd_Date DATE NOT NULL,
	nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
	nUser_Update INT,
    dDateTime_Update DATETIME,
	nUser_Delete INT,
    dDateTime_Delete DATETIME
);

CREATE TABLE Form_Type (
    nId_Form_Type INT IDENTITY(1,1) PRIMARY KEY,
	sName NVARCHAR(500) NOT NULL,
	nId_FormType_Padre INT NOT NULL,
	nId_Form_SubType INT NOT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
	nUser_Update INT,
    dDateTime_Update DATETIME,
	nUser_Delete INT,
    dDateTime_Delete DATETIME
);

CREATE TABLE Forms (
    nId_Form INT IDENTITY(1,1) PRIMARY KEY,
	sName NVARCHAR(500) NOT NULL,
    sDescription NVARCHAR(MAX) NOT NULL,
    nId_Evaluation INT NOT NULL,
    nId_Form_Type INT NOT NULL,
	nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
	nUser_Update INT,
    dDateTime_Update DATETIME,
	nUser_Delete INT,
    dDateTime_Delete DATETIME,
	FOREIGN KEY (nId_Evaluation) REFERENCES Evaluations(nId_Evaluation),
	FOREIGN KEY (nId_Form_Type) REFERENCES Form_Type(nId_Form_Type)
);


CREATE TABLE Evaluators (
    nId_Evaluator INT NOT NULL,
	nId_Assessed INT NOT NULL,
    nId_Form INT NOT NULL,
	FOREIGN KEY (nId_Evaluator) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Assessed) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Form) REFERENCES Forms(nId_Form)
);


CREATE TABLE Qualified_Evaluations (
    nId_Qualified_Evaluation INT IDENTITY(1,1) PRIMARY KEY,
	nId_Form INT NOT NULL,
	nId_Evaluator INT NOT NULL,
	nId_Assessed INT NOT NULL,
	nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
	nUser_Update INT,
    dDateTime_Update DATETIME,
	nUser_Delete INT,
    dDateTime_Delete DATETIME,
	FOREIGN KEY (nId_Evaluator) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Assessed) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Form) REFERENCES Forms(nId_Form)
);

-----------------------------

CREATE TABLE Competence (
    nId_Competence INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
    sName NVARCHAR(100) NOT NULL,
    sDescription NVARCHAR(255) NULL,
    sCode NVARCHAR(50) NOT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL
);

/*CREATE TABLE Evaluation_Configuration (
    nId_Evaluation_Configuration INT PRIMARY KEY NOT NULL,
    nId_Competence INT NOT NULL,
    nId_Form INT NOT NULL,
    nPercentage VARCHAR(50) NOT NULL,
    nId_Rol INT NOT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL,
    FOREIGN KEY (nId_Competence) REFERENCES Competence(nId_Competence),
    FOREIGN KEY (nId_Form) REFERENCES Form(nId_Form)
);*/

CREATE TABLE Questions (
    nId_Question INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
    nId_Form INT NOT NULL,
    sText NVARCHAR(255) NOT NULL,
    nId_Type_Question INT NOT NULL,
    bRequired BIT NOT NULL,
    sPlaceHolder NVARCHAR(100),
    nId_Competence INT NOT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL,
    FOREIGN KEY (nId_Form) REFERENCES Forms(nId_Form),
    FOREIGN KEY (nId_Competence) REFERENCES Competence(nId_Competence)
);

CREATE TABLE Answer (
    nId_Answer INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
    nId_Qualified_Evaluation INT NOT NULL,
    nId_Question INT NOT NULL,
    sText_Answer NVARCHAR(255) NULL,
    nNumber_Answer INT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL,
    FOREIGN KEY (Qualified_Evaluation) REFERENCES Questions(Qualified_Evaluation),
    FOREIGN KEY (nId_Question) REFERENCES Questions(nId_Question)
);
```