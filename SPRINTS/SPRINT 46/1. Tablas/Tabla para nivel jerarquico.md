```SQL
CREATE TABLE Job_Category (
    nId_Job_Category INT IDENTITY(1,1) PRIMARY KEY,
    sName NVARCHAR(500) NOT NULL,
    sDescription NVARCHAR(500) NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL,
);

CREATE TABLE Hierarchy_Level (
    nId_Hierarchy_Level INT IDENTITY(1,1) PRIMARY KEY,
    sName NVARCHAR(500) NOT NULL,
    sDescription NVARCHAR(500) NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT NULL,
    dDateTime_Update DATETIME NULL,
    nUser_Delete INT NULL,
    dDateTime_Delete DATETIME NULL,
);

CREATE TABLE Job_Competency_Weight (
    nId_Job_Competency_Weight INT IDENTITY(1,1) PRIMARY KEY,
    nId_Competence INT NOT NULL,
    nId_Job_Category INT NOT NULL,
    nPercentage DECIMAL(4,3) NOT NULL,
    nMax_Value INT NOT NULL,
    nExpected_value DECIMAL(4,3) NOT NULL,
	nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT,
    dDateTime_Update DATETIME,
    nUser_Delete INT,
    dDateTime_Delete DATETIME,
    CONSTRAINT chk_nPercentage CHECK (nPercentage BETWEEN 0 AND 1),
	FOREIGN KEY (nId_Competence) REFERENCES Competence(nId_Competence),
	FOREIGN KEY (nId_Job_Category) REFERENCES Job_Category(nId_Job_Category)
);

CREATE TABLE Job_TypeForm_Weight (
    nId_Job_TypeForm_Weight INT IDENTITY(1,1) PRIMARY KEY,
    nId_Hierarchy_Level INT NOT NULL,
    nId_Job_Category INT NOT NULL,
    nPercentage DECIMAL(4,3) NOT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT,
    dDateTime_Update DATETIME,
    nUser_Delete INT,
    dDateTime_Delete DATETIME,
    CONSTRAINT chk_nPercentage_Job_Typeform_weight CHECK (nPercentage BETWEEN 0 AND 1),
	FOREIGN KEY (nId_Hierarchy_Level) REFERENCES Hierarchy_Level(nId_Hierarchy_Level)
	FOREIGN KEY (nId_Job_Category) REFERENCES Job_Category(nId_Job_Category)
);

CREATE TABLE Competence_Average_Evaluator (
    nId_Competence_Average_Evaluator INT IDENTITY(1,1) PRIMARY KEY,
    nId_Assessed INT NOT NULL,
    nId_Evaluator INT NOT NULL,
    nId_Evaluation INT NOT NULL,
	nId_Hierarchy_Level INT NOT NULL,
    nId_Job_Competency_Weight INT NOT NULL,
    nValue_Average DECIMAL(4,2) NOT NULL,
    nValue_Average_Weight DECIMAL(4,2) NOT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT,
    dDateTime_Update DATETIME,
    nUser_Delete INT,
    dDateTime_Delete DATETIME,
	FOREIGN KEY (nId_Hierarchy_Level) REFERENCES Hierarchy_Level(nId_Hierarchy_Level),
	FOREIGN KEY (nId_Assessed) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Evaluator) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Evaluation) REFERENCES Evaluations(nId_Evaluation),
	FOREIGN KEY (nId_Job_Competency_Weight) REFERENCES Job_Competency_Weight(nId_Job_Competency_Weight)
);

CREATE TABLE Competence_Average (
    nId_Competence_Average INT IDENTITY(1,1) PRIMARY KEY,
    nId_Assessed INT NOT NULL,
    nId_Evaluation INT NOT NULL,
    nId_Job_Competency_Weight INT NOT NULL,
    nValue DECIMAL(4,2) NOT NULL,
    nValue_Percentage DECIMAL(4,3) NOT NULL,
    nUser_Creator INT NOT NULL,
    dDateTime_Creator DATETIME NOT NULL,
    nUser_Update INT,
    dDateTime_Update DATETIME,
    nUser_Delete INT,
    dDateTime_Delete DATETIME,
	FOREIGN KEY (nId_Assessed) REFERENCES Colaboradores(nId_Colaborador),
	FOREIGN KEY (nId_Evaluation) REFERENCES Evaluations(nId_Evaluation),
	FOREIGN KEY (nId_Job_Competency_Weight) REFERENCES Job_Competency_Weight(nId_Job_Competency_Weight)
);

ALTER TABLE Qualified_Evaluations
ADD nTotal_Score DECIMAL(6,4)