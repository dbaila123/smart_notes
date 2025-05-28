```SQL
-- Tablas de primer nivel

CREATE TABLE Evaluations (
    nId_Evaluation INT IDENTITY(1,1) PRIMARY KEY,
    sName NVARCHAR(500) NOT NULL,
    nState INT NOT NULL,
    sDescription NVARCHAR(MAX),
    dInit_Date DATE NOT NULL,
    dEnd_Date DATE NOT NULL,
    bIsNotified BIT DEFAULT 0,
    dShare_Results DATETIME,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL
);

CREATE TABLE Form_Type (
    nId_Form_Type INT IDENTITY(1,1) PRIMARY KEY,
    sName NVARCHAR(500) NOT NULL,
    nId_FormType_Padre INT NULL,
    sIcon NVARCHAR(100) NULL,
    sDescription NVARCHAR(500) NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL,
    FOREIGN KEY (nId_FormType_Padre) REFERENCES Form_Type(nId_Form_Type)
);

CREATE TABLE Form_Template (
    nId_Form_Template INT IDENTITY(1,1) PRIMARY KEY,
    sName NVARCHAR(255) NOT NULL,
    sDescription NVARCHAR(255) NULL,
    nUser_Creator INT NOT NULL,
    dDatetime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDatetime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDatetime_Delete DATETIME NULL 
);

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

CREATE TABLE Type_Question (
    nId_Type_Question INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
    sName NVARCHAR(255) NOT NULL,
    nUser_Creator INT NOT NULL,
    dDatetime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDatetime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDatetime_Delete DATETIME NULL
);

CREATE TYPE AssessedFormTypeList AS TABLE(
    nId_Assessed INT,
    nId_FormType INT
);

-- Tablas de segundo nivel

CREATE TABLE Forms (
    nId_Form INT IDENTITY(1,1) PRIMARY KEY,
    sName NVARCHAR(500) NOT NULL,
    sDescription NVARCHAR(MAX) NOT NULL,
    nId_Evaluation INT NOT NULL,
    nId_Form_Type INT NOT NULL,
    nId_Form_SubType INT NULL,
    nId_Form_Template INT NOT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL,
    FOREIGN KEY (nId_Evaluation) REFERENCES Evaluations(nId_Evaluation) ON DELETE CASCADE,
    FOREIGN KEY (nId_Form_Type) REFERENCES Form_Type(nId_Form_Type),
    FOREIGN KEY (nId_Form_SubType) REFERENCES Form_Type(nId_Form_Type),
    FOREIGN KEY (nId_Form_Template) REFERENCES Form_Template(nId_Form_Template)
);

CREATE TABLE Questions_Template (
    nId_Question_Template INT IDENTITY(1,1) PRIMARY KEY NOT NULL, -- Corregido el nombre
    nId_Form_Template INT NOT NULL,
    sText NVARCHAR(255) NOT NULL,
    nId_Type_Question INT NOT NULL,
    bRequired BIT NOT NULL,
    sPlaceHolder NVARCHAR(255) NULL,
    nId_Competence INT NULL,
    nUser_Creator INT NOT NULL,
    dDatetime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDatetime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDatetime_Delete DATETIME NULL,
    FOREIGN KEY (nId_Form_Template) REFERENCES Form_Template(nId_Form_Template),
    FOREIGN KEY (nId_Type_Question) REFERENCES Type_Question(nId_Type_Question),
    FOREIGN KEY (nId_Competence) REFERENCES Competence(nId_Competence)
);

CREATE TABLE Evaluators (
    nId_Evaluator INT NOT NULL,
    nId_Assessed INT NOT NULL,
    nId_Form INT NOT NULL,
    PRIMARY KEY (nId_Evaluator, nId_Assessed, nId_Form), -- Añadida clave primaria compuesta
    FOREIGN KEY (nId_Evaluator) REFERENCES Colaboradores(nId_Colaborador),
    FOREIGN KEY (nId_Assessed) REFERENCES Colaboradores(nId_Colaborador),
    FOREIGN KEY (nId_Form) REFERENCES Forms(nId_Form) ON DELETE CASCADE
);

CREATE TABLE Qualified_Evaluations (
    nId_Qualified_Evaluation INT IDENTITY(1,1) PRIMARY KEY,
    nId_Form INT NOT NULL,
    nId_Evaluator INT NOT NULL,
    nId_Assessed INT NOT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL,
    FOREIGN KEY (nId_Evaluator) REFERENCES Colaboradores(nId_Colaborador),
    FOREIGN KEY (nId_Assessed) REFERENCES Colaboradores(nId_Colaborador),
    FOREIGN KEY (nId_Form) REFERENCES Forms(nId_Form)
);

-- Tablas de tercer nivel

CREATE TABLE Questions (
    nId_Question INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
    nId_Form INT NOT NULL,
    sText NVARCHAR(255) NOT NULL,
    nId_Type_Question INT NOT NULL,
    bRequired BIT NOT NULL,
    sPlaceHolder NVARCHAR(100) NULL,
    nId_Competence INT NULL,
    nId_Question_Template INT NULL, -- Hecho nullable y corregido el nombre
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL,
    FOREIGN KEY (nId_Form) REFERENCES Forms(nId_Form) ON DELETE CASCADE,
    FOREIGN KEY (nId_Competence) REFERENCES Competence(nId_Competence),
    FOREIGN KEY (nId_Type_Question) REFERENCES Type_Question(nId_Type_Question),
    FOREIGN KEY (nId_Question_Template) REFERENCES Questions_Template(nId_Question_Template)
);

CREATE TABLE dataList_Template (
    nId_List INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
    nId_Question_Template INT NOT NULL, -- Corregido el nombre
    sText NVARCHAR(255) NOT NULL,
    nValue INT NOT NULL,
    nUser_Creator INT NOT NULL,
    dDatetime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDatetime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDatetime_Delete DATETIME NULL,
    FOREIGN KEY (nId_Question_Template) REFERENCES Questions_Template(nId_Question_Template)
);

CREATE TABLE data_list (
    nId_Data_List INT IDENTITY(1,1) PRIMARY KEY NOT NULL,
    nId_Question INT NOT NULL,
    sText NVARCHAR(255) NOT NULL,
    nValue INT NOT NULL,
    nUser_Creator INT NOT NULL,
    dDatetime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDatetime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDatetime_Delete DATETIME NULL,
    FOREIGN KEY (nId_Question) REFERENCES Questions(nId_Question) ON DELETE CASCADE
);

-- Tabla de cuerto nivel

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
    FOREIGN KEY (nId_Qualified_Evaluation) REFERENCES Qualified_Evaluations(nId_Qualified_Evaluation), -- Corregido
    FOREIGN KEY (nId_Question) REFERENCES Questions(nId_Question)
);

-- Tabla de relación

CREATE TABLE Data_List_Answer (
    nId_Data_List INT NOT NULL,
    nId_Answer INT NOT NULL,
    PRIMARY KEY (nId_Data_List, nId_Answer), -- Añadida clave primaria compuesta
    FOREIGN KEY (nId_Data_List) REFERENCES data_list(nId_Data_List),
    FOREIGN KEY (nId_Answer) REFERENCES Answer(nId_Answer)
);