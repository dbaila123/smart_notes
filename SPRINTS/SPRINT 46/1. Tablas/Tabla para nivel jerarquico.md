```SQL
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