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
-----------------------------

CREATE TABLE Competence (
    nId_Competence INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
    sName VARCHAR(100) NOT NULL,
    sDescription VARCHAR(255) NULL,
    sCode VARCHAR(50) NOT NULL,
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
    sText VARCHAR(255) NOT NULL,
    sType VARCHAR(50) NOT NULL,
    bRequired BIT NOT NULL,
    sPlaceHolder VARCHAR(100) NOT NULL,
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
    sText_Answer VARCHAR(255) NULL,
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