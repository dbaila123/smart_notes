```SQL
CREATE PROCEDURE sp_smart_evaluation_getFormTypeAndTemplate
AS
BEGIN
    SET NOCOUNT ON;
    
    SELECT 
        nId_Form_Type AS nId,
        sName,
        sIcon
    FROM Form_Type
    WHERE nId_FormType_Padre IS NULL;
    
    -- Obtener los subtipos con su padre
    SELECT 
        ft.nId_Form_Type AS nId,
        ft.sName,
        ft.nId_FormType_Padre AS nIdParent
    FROM Form_Type ft
    WHERE ft.nId_FormType_Padre IS NOT NULL;
    
    /*SELECT 
        nId_Form_Template AS nId,
        sName,
        sDescription
    FROM Form_Template;*/
END