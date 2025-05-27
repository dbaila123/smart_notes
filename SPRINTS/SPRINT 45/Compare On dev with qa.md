```
revoke select on Access_Service_Token from rol_QA  
go  
  
alter table Answer  
    alter column sText_Answer varchar(255) null  
go  
  
revoke select on Aprobaciones from rol_QA  
go  
  
revoke select on Aprobadores from rol_QA  
go  
  
revoke select on Areas from rol_QA  
go  
  
revoke select on Asistencia_Historicos from rol_QA  
go  
  
revoke select on Asistencias from rol_QA  
go  
  
revoke select on Attendances from rol_QA  
go  
  
revoke select on Biometrico_Log from rol_QA  
go  
  
revoke select on Cargos from rol_QA  
go  
  
revoke select on Cargos_Colaboradores from rol_QA  
go  
  
revoke select on Categorias from rol_QA  
go  
  
revoke select on Categorias_Permisos from rol_QA  
go  
  
revoke select on Celebracion from rol_QA  
go  
  
revoke select on Cierre_Horas from rol_QA  
go  
  
revoke select on Clientes from rol_QA  
go  
  
revoke select on ClosingDate from rol_QA  
go  
  
revoke select on Colaboradores from rol_QA  
go  
  
revoke select on Colaboradores_Logs from rol_QA  
go  
  
revoke select on Colaboradores_Proyectos from rol_QA  
go  
  
alter table Competence  
    alter column sName varchar(100) not null  
go  
  
alter table Competence  
    alter column sDescription varchar(255) null  
go  
  
alter table Competence  
    alter column sCode varchar(50) not null  
go  
  
revoke select on Configs from rol_QA  
go  
  
revoke select on Contactos from rol_QA  
go  
  
revoke select on Contract_Templates from rol_QA  
go  
  
revoke select on Contratos from rol_QA  
go  
  
revoke select on Coordinadores from rol_QA  
go  
  
revoke select on Coordinadores_Proyectos from rol_QA  
go  
  
revoke select on Correos_Programados from rol_QA  
go  
  
revoke select on Cuentas_Bancarias from rol_QA  
go  
  
alter table Data_List_Answer  
    drop constraint PK__Data_Lis__98EFA94A41E8EA26  
go  
  
revoke select on DerechoHabientes from rol_QA  
go  
  
revoke select on Direcciones from rol_QA  
go  
  
revoke select on Documentos from rol_QA  
go  
  
revoke select on Emails from rol_QA  
go  
  
revoke select on Empresas_Usuarias from rol_QA  
go  
  
revoke select on Entidades_Transacciones from rol_QA  
go  
  
revoke select on Especialidades from rol_QA  
go  
  
alter table Evaluations  
    add bIsNotified bit  
        constraint DF_Evaluation_Notified default 0  
go  
  
alter table Evaluators  
    drop constraint PK__Evaluato__8CB6D93896A898F9  
go  
  
alter table Evaluators  
    add constraint FK_Evaluator_Form  
        foreign key (nId_Form) references Forms  
            on delete cascade  
go  
  
alter table Evaluators  
    drop constraint FK__Evaluator__nId_F__1E105D02  
go  
  
revoke select on Feriados from rol_QA  
go  
  
revoke select on Fichas_Sistematologicas from rol_QA  
go  
  
revoke select on Files from rol_QA  
go  
  
alter table Form_Template  
    alter column sName varchar(255) not null  
go  
  
alter table Form_Template  
    alter column sDescription varchar(255) null  
go  
  
alter table Form_Template  
    drop column nUser_Creator  
go  
  
-- column reordering is not supported Form_Type.nId_FormType_Padre  
  
-- column reordering is not supported Form_Type.sIcon  
  
-- column reordering is not supported Form_Type.sDescription  
  
alter table Form_Type  
    drop constraint FK__Form_Type__nId_F__0915401C  
go  
  
alter table Forms  
    alter column nId_Form_Type int null  
go  
  
-- column reordering is not supported Forms.nId_Form_Type  
  
-- column reordering is not supported Forms.nId_Form_SubType  
  
alter table Forms  
    alter column nId_Form_Template int null  
go  
  
-- column reordering is not supported Forms.nId_Form_Template  
  
alter table Forms  
    add constraint FK_Forms_Evaluation  
        foreign key (nId_Evaluation) references Evaluations  
            on delete cascade  
go  
  
alter table Forms  
    drop constraint FK__Forms__nId_Evalu__11AA861D  
go  
  
alter table Forms  
    drop constraint FK__Forms__nId_Form___129EAA56  
go  
  
alter table Forms  
    drop constraint FK__Forms__nId_Form___1392CE8F  
go  
  
alter table Forms  
    drop constraint FK__Forms__nId_Form___1486F2C8  
go  
  
revoke select on GetConfiguracionesDto from rol_QA  
go  
  
revoke select on GetTareoStatusXColaboradorDto from rol_QA  
go  
  
revoke select on Giros_Negocios from rol_QA  
go  
  
revoke select on History_Closing_Transactions from rol_QA  
go  
  
revoke select on History_Hours_Closing from rol_QA  
go  
  
revoke select on Horarios from rol_QA  
go  
  
revoke select on Horarios_Colaboradores from rol_QA  
go  
  
revoke select on Horarios_Detalles from rol_QA  
go  
  
revoke select on HtmlTemplate from rol_QA  
go  
  
revoke select on Justificaciones_Historicos from rol_QA  
go  
  
revoke select on ListaAsistenciaQuery from rol_QA  
go  
  
revoke select on ListaFichaQuery from rol_QA  
go  
  
revoke select on Log_Errors from rol_QA  
go  
  
revoke select on MailingRecipient from rol_QA  
go  
  
revoke select on MailingTemplate from rol_QA  
go  
  
revoke select on Marcas from rol_QA  
go  
  
revoke select on Marcas_test from rol_QA  
go  
  
revoke select on Metodologias from rol_QA  
go  
  
revoke select on Metodologias_Categorias from rol_QA  
go  
  
revoke select on Modulos from rol_QA  
go  
  
revoke select on Notificaciones from rol_QA  
go  
  
revoke select on Notificaciones_Logs from rol_QA  
go  
  
revoke select on Notificaciones_test from rol_QA  
go  
  
revoke select on OSolicitation from rol_QA  
go  
  
revoke select on OTask from rol_QA  
go  
  
revoke select on Observaciones from rol_QA  
go  
  
revoke select on Ocupaciones from rol_QA  
go  
  
revoke select on Opciones from rol_QA  
go  
  
revoke select on Paises from rol_QA  
go  
  
revoke select on PasswordResetTokens from rol_QA  
go  
  
revoke select on Permisos from rol_QA  
go  
  
revoke select on Permisos_Opciones from rol_QA  
go  
  
revoke select on Personas from rol_QA  
go  
  
revoke select on Planes from rol_QA  
go  
  
revoke select on PreferenciasColaborador from rol_QA  
go  
  
revoke select on ProcedureLog from rol_QA  
go  
  
revoke select on Proyecto_Lider from rol_QA  
go  
  
revoke select on Proyecto_Requerimiento_Colaborador_Notificacion from rol_QA  
go  
  
revoke select on Proyectos from rol_QA  
go  
  
revoke select on PublicFile from rol_QA  
go  
  
revoke select on Push_Tokens from rol_QA  
go  
  
alter table Questions  
    alter column sText varchar(255) not null  
go  
  
alter table Questions  
    alter column nId_Type_Question int null  
go  
  
alter table Questions  
    alter column sPlaceHolder varchar(100) null  
go  
  
alter table Questions  
    alter column nId_Competence int not null  
go  
  
-- column reordering is not supported Questions.nId_Question_Template  
  
alter table Questions  
    add constraint FK_Question_Form  
        foreign key (nId_Form) references Forms  
            on delete cascade  
go  
  
alter table Questions  
    drop constraint FK__Questions__nId_C__26A5A303  
go  
  
alter table Questions  
    drop constraint FK__Questions__nId_F__25B17ECA  
go  
  
alter table Questions  
    drop constraint FK__Questions__nId_Q__288DEB75  
go  
  
alter table Questions  
    drop constraint FK__Questions__nId_T__2799C73C  
go  
  
alter table Questions  
    add foreign key (nId_Competence) references Competence  
go  
  
alter table Questions_Template  
    alter column sText varchar(255) not null  
go  
  
alter table Questions_Template  
    alter column sPlaceHolder varchar(255) null  
go  
  
alter table Questions_Template  
    alter column nId_Competence int not null  
go  
  
revoke select on Regimen from rol_QA  
go  
  
revoke select on Requerimientos from rol_QA  
go  
  
revoke select on Requerimientos_Categoria from rol_QA  
go  
  
revoke select on Requerimientos_Proyectos_Historicos from rol_QA  
go  
  
revoke select on RequestWorkflowStep from rol_QA  
go  
  
revoke select on Responsabilidades from rol_QA  
go  
  
revoke select on Revision_Tareas from rol_QA  
go  
  
revoke select on Revisiones from rol_QA  
go  
  
revoke select on Roles from rol_QA  
go  
  
revoke select on Roles_Usuarios from rol_QA  
go  
  
revoke select on Rubros from rol_QA  
go  
  
revoke select on Sedes from rol_QA  
go  
  
revoke select on Sedes_Colaboradores from rol_QA  
go  
  
revoke select on Sesiones from rol_QA  
go  
  
revoke select on Solicitud_Tesoreria from rol_QA  
go  
  
revoke select on Solicitud_Tesoreria_Historico from rol_QA  
go  
  
revoke select on Solicitudes from rol_QA  
go  
  
revoke select on Solicitudes_Audit from rol_QA  
go  
  
revoke select on Solicitudes_Historicos from rol_QA  
go  
  
revoke select on Solicitudes_Historicos_TEST from rol_QA  
go  
  
revoke select on Solicitudes_Reemplazo from rol_QA  
go  
  
revoke select on Solicitudes_Tesoreria_Participantes from rol_QA  
go  
  
revoke select on SubdomainDatabaseMapping from rol_QA  
go  
  
revoke select on Supervisor_Proyecto from rol_QA  
go  
  
revoke select on Tags_Entities from rol_QA  
go  
  
revoke select on Tareas from rol_QA  
go  
  
revoke select on Tareas_Historicos from rol_QA  
go  
  
revoke select on TaskListIncompleteQuery from rol_QA  
go  
  
revoke select on Tb_log_procedure_changes from rol_QA  
go  
  
revoke select on Team from rol_QA  
go  
  
revoke select on Team_Colaboradors from rol_QA  
go  
  
revoke select on Telefonos from rol_QA  
go  
  
revoke select on Tema_Colores from rol_QA  
go  
  
revoke select on Tema_Fondos from rol_QA  
go  
  
revoke select on Temas from rol_QA  
go  
  
revoke select on Tesoreria_Categorias from rol_QA  
go  
  
revoke select on Tesoreria_Cuotas from rol_QA  
go  
  
revoke select on Tesoreria_Cuotas_Consolidado from rol_QA  
go  
  
revoke select on Tesoreria_Tipos_Solicitud from rol_QA  
go  
  
revoke select on TipoMarcas_HorarioDetalles from rol_QA  
go  
  
revoke select on Tipos_Motivos from rol_QA  
go  
  
revoke select on Tipos_Solicitudes from rol_QA  
go  
  
revoke select on Titulo_Version from rol_QA  
go  
  
revoke select on Transacciones_Saldo_Mins from rol_QA  
go  
  
alter table Type_Question  
    alter column sName varchar(255) not null  
go  
  
revoke select on Ubigeos from rol_QA  
go  
  
revoke select on UserBrowser from rol_QA  
go  
  
revoke select on UserQuery from rol_QA  
go  
  
revoke select on Usuarios from rol_QA  
go  
  
revoke select on VacacionesAsignacionesLog from rol_QA  
go  
  
revoke select on Version from rol_QA  
go  
  
revoke select on WorkflowByRequestType from rol_QA  
go  
  
revoke select on Workflows from rol_QA  
go  
  
revoke select on Xample from rol_QA  
go  
  
revoke select on cantidad_celebrations from rol_QA  
go  
  
revoke select on cargos_test from rol_QA  
go  
  
revoke select on cierreAsistenciaReportes from rol_QA  
go  
  
revoke select on cierreAsistencias from rol_QA  
go  
  
revoke select on closure_Request from rol_QA  
go  
  
revoke select on colaboradores_supervisor from rol_QA  
go  
  
revoke select on contratos_Historicos from rol_QA  
go  
  
alter table dataList_Template  
    drop constraint FK__dataList___nId_Q__2B6A5820  
go  
  
alter table Questions_Template  
    drop constraint PK__Question__E531739E797D5A34  
go  
  
alter table Questions_Template  
    add nId_Question int identity  
        primary keygo  
  
-- column reordering is not supported Questions_Template.nId_Question  
  
alter table Questions_Template  
    drop column nId_Question_Template  
go  
  
alter table dataList_Template  
    add nId_Question int not null  
        references Questions_Template  
go  
  
-- column reordering is not supported dataList_Template.nId_Question  
  
alter table dataList_Template  
    drop column nId_Question_Template  
go  
  
alter table dataList_Template  
    alter column sText varchar(255) not null  
go  
  
alter table dataList_Template  
    alter column nValue int null  
go  
  
alter table data_list  
    alter column sText varchar(255) null  
go  
  
alter table data_list  
    alter column nValue int null  
go  
  
alter table data_list  
    add constraint FK_DataList_Question  
        foreign key (nId_Question) references Questions  
            on delete cascade  
go  
  
alter table data_list  
    drop constraint FK__data_list__nId_Q__2E46C4CB  
go  
  
revoke select on dispositivo_Informacion from rol_QA  
go  
  
revoke select on justificaciones_Proyectos_Requerimientos from rol_QA  
go  
  
revoke select on pryRqCatMinutosCalculados from rol_QA  
go  
  
revoke select on pryRqMinutosCalculados from rol_QA  
go  
  
revoke select on questionsAndAnswers from rol_QA  
go  
  
revoke select on saldos_Vacaciones_Colaboradores from rol_QA  
go  
  
revoke select on saldos_Vacaciones_Colaboradores_test from rol_QA  
go  
  
revoke select on socketLogs from rol_QA  
go  
  
revoke select on solicitudes_rollback from rol_QA  
go  
  
create table sysdiagrams  
(  
    name         sysname not null,  
    principal_id int     not null,  
    diagram_id   int identity  
        primary key,  
    version      int,  
    definition   varbinary(max),  
    constraint UK_principal_name  
        unique (principal_id, name)  
)  
go  
  
exec sp_addextendedproperty 'microsoft_database_tools_support', 1, 'SCHEMA', 'dbo', 'TABLE', 'sysdiagrams'  
go  
  
revoke select on tarea_rollback from rol_QA  
go  
  
revoke select on userConnections from rol_QA  
go  
  
revoke select on Horario_Total_Detalle from rol_QA  
go  
  
revoke select on collaborators_hours_LOOKER from rol_QA  
go  
  
revoke select on personas_colaboradores from rol_QA  
go  
  
revoke select on personas_usuarios from rol_QA  
go  
  
revoke select on sp_smart_detail_clousure_request_listado from rol_QA  
go  
  
revoke select on v_Cierre_Asistencias_Detalle from rol_QA  
go  
  
revoke select on v_List_Programmed_Emails from rol_QA  
go  
  
revoke select on v_Listado_Aprobadores_Solicitudes from rol_QA  
go  
  
revoke select on v_Listado_Asistencias from rol_QA  
go  
  
revoke select on v_Listado_Asistencias_Reporte from rol_QA  
go  
  
revoke select on v_Listado_Asistencias_new from rol_QA  
go  
  
revoke select on v_Listado_Asistencias_vista_cliente from rol_QA  
go  
  
revoke select on v_Listado_Campus from rol_QA  
go  
  
revoke select on v_Listado_Cargos from rol_QA  
go  
  
revoke select on v_Listado_Cierre_Asistencias from rol_QA  
go  
  
revoke select on v_Listado_Cierre_Asistencias_Reporte from rol_QA  
go  
  
revoke select on v_Listado_Clientes_MG from rol_QA  
go  
  
revoke select on v_Listado_Colaboradores_Tareo_Incompleto from rol_QA  
go  
  
revoke select on v_Listado_Coordinators from rol_QA  
go  
  
revoke select on v_Listado_Historico_Cierre_Asistencias from rol_QA  
go  
  
revoke select on v_Listado_Historico_Solicitudes from rol_QA  
go  
  
revoke select on v_Listado_Historicos_Compensacion_Recuperacion_Permisos from rol_QA  
go  
  
revoke select on v_Listado_Horarios from rol_QA  
go  
  
revoke select on v_Listado_Mailing_Template from rol_QA  
go  
  
revoke select on v_Listado_Notificaciones from rol_QA  
go  
  
revoke select on v_Listado_Plantillas_Contratos from rol_QA  
go  
  
revoke select on v_Listado_Proyecto_Requerimientos from rol_QA  
go  
  
revoke select on v_Listado_Proyectos from rol_QA  
go  
  
revoke select on v_Listado_Proyectos_con_requerimientos from rol_QA  
go  
  
revoke select on v_Listado_Proyectos_refact from rol_QA  
go  
  
revoke select on v_Listado_Requerimientos from rol_QA  
go  
  
revoke select on v_Listado_Requerimientos_refact from rol_QA  
go  
  
revoke select on v_Listado_Roles from rol_QA  
go  
  
revoke select on v_Listado_Saldos_Horas from rol_QA  
go  
  
revoke select on v_Listado_Solicitudes from rol_QA  
go  
  
revoke select on v_Listado_Solicitudes_Reporte from rol_QA  
go  
  
revoke select on v_Listado_Solicitudes_TEST from rol_QA  
go  
  
revoke select on v_Listado_Tardanzas_Cierre from rol_QA  
go  
  
revoke select on v_Listado_Tardanzas_Cierre_Report from rol_QA  
go  
  
revoke select on v_Listado_Tareas from rol_QA  
go  
  
revoke select on v_Listado_Tareas_Calidad from rol_QA  
go  
  
revoke select on v_Listado_Tareas_Cliente_new from rol_QA  
go  
  
revoke select on v_Listado_Tareas_Reporte from rol_QA  
go  
  
revoke select on v_Listado_Tareas_dashboard from rol_QA  
go  
  
revoke select on v_Listado_Tareas_david from rol_QA  
go  
  
revoke select on v_Listado_Tareo_Estado_Colaboradores from rol_QA  
go  
  
revoke select on v_Listado_Tesoreria from rol_QA  
go  
  
revoke select on v_Listado_Tesoreria_Report from rol_QA  
go  
  
revoke select on v_Listado_Usuarios from rol_QA  
go  
  
revoke select on v_Listado_request_transaction from rol_QA  
go  
  
revoke select on v_Listado_ultimas_marcas from rol_QA  
go  
  
revoke select on v_Participantes_Tesoreria_Principales from rol_QA  
go  
  
revoke select on v_Supervisores_Asginados from rol_QA  
go  
  
revoke select on v_all_Collaborator_Organigrama from rol_QA  
go  
  
revoke select on v_celebraciones from rol_QA  
go  
  
revoke select on v_dashboard_asistencias from rol_QA  
go  
  
revoke select on v_dashboard_asistencias_calculations from rol_QA  
go  
  
revoke select on v_dashboard_ceses_altas from rol_QA  
go  
  
revoke select on v_dashboard_distribucion_colaboradores from rol_QA  
go  
  
revoke select on v_dashboard_num_solicitudes from rol_QA  
go  
  
revoke select on v_horario_dias_total_horas from rol_QA  
go  
  
-- dbo.v_list_evaluation_resume source  
  
alter VIEW v_list_evaluation_resume  
AS  
    WITH EvaluationsResults AS (  
           SELECT  
               ec.nId_Evaluation,  
               COUNT(CASE WHEN ec.nAnswered = 1 THEN 1 END) + COUNT(CASE WHEN ec.nAnswered = 0 THEN 1 END) AS Expected,  
               COUNT(CASE WHEN ec.nAnswered = 1 THEN 1 END) AS nAnswered  
           FROM (  
               select  
                   e.nId_Evaluator,  
                   e.nId_Evaluation,  
                   MIN(e.nAnswered) AS nAnswered  
               from v_list_evaluators_form_completition_status e  
               group by e.nId_Evaluator,  
                   e.nId_Evaluation  
           ) ec  
           GROUP BY ec.nId_Evaluation  
       )  
       SELECT  
           e.nId_Evaluation,  
           e.sName AS sEvaluation_Name,  
          e.sDescription AS sEvaluation_Description,  
           e.dInit_Date AS dStart_Date,  
           e.dEnd_Date AS dEnd_Date,  
          e.dDateTime_Creator,  
          (  
             SELECT  
                *  
             FROM (  
                SELECT DISTINCT ft.sIcon, ft.sName  
                FROM Form_Type ft  
                JOIN Forms f1 on ft.nId_Form_Type = f1.nId_Form_Type  
                WHERE f1.nId_Evaluation = e.nId_Evaluation) as s  
             FOR JSON PATH          ) AS sIcons,  
          CASE  
             WHEN e.nState = 1 THEN 1  
             WHEN DATEADD(HOUR, -5, GETDATE()) BETWEEN e.dInit_Date AND e.dEnd_Date THEN 2  
             WHEN DATEADD(HOUR, -5, GETDATE()) < e.dInit_Date THEN 4  
             WHEN DATEADD(HOUR, -5, GETDATE()) > e.dEnd_Date THEN 3  
          END AS nState,  
           p.sPersona_Nombre AS sCreator,  
           CAST(er.nAnswered AS VARCHAR) + '/' + CAST(er.Expected AS VARCHAR) AS sParticipants,  
          ROUND(CAST(er.nAnswered AS FLOAT) / er.Expected, 4) * 100 AS nProgress  
       FROM Evaluations e  
       JOIN Usuarios u ON e.nUser_Creator = u.nId_Usuario  
       JOIN Personas p ON u.nId_Persona = p.nId_Persona  
       JOIN Forms f ON e.nId_Evaluation = f.nId_Evaluation  
       JOIN Evaluators eva ON f.nId_Form = eva.nId_Form  
       JOIN EvaluationsResults er ON e.nId_Evaluation = er.nId_Evaluation  
       GROUP BY  
          e.nId_Evaluation,  
          e.sName,  
          e.dInit_Date,  
          e.dEnd_Date,  
          p.sPersona_Nombre,  
          er.nAnswered,  
          er.Expected,  
          e.sDescription,  
          e.nState,  
          e.dDateTime_Creator;  
go  
  
revoke select on v_listado_Colaborador from rol_QA  
go  
  
revoke select on v_listado_Colaborador_report from rol_QA  
go  
  
revoke select on v_listado_Colaborador_reporte from rol_QA  
go  
  
revoke select on v_listado_Colaborador_test from rol_QA  
go  
  
revoke select on v_listado_Modulo_Configuraciones from rol_QA  
go  
  
revoke select on v_listado_Transacciones_Saldo_Mins from rol_QA  
go  
  
revoke select on v_listado_contratos from rol_QA  
go  
  
revoke select on v_listado_emails_personas from rol_QA  
go  
  
revoke select on v_listado_saldos_vacaciones from rol_QA  
go  
  
revoke select on v_listado_tareas_semanales from rol_QA  
go  
  
revoke select on v_persona_empresa from rol_QA  
go  
  
revoke select on v_reporte_asitencias_diarias from rol_QA  
go  
  
revoke select on vw_user_person_collab from rol_QA  
go  
  
revoke execute on Function_Get_Entry_Mark from rol_QA  
go  
  
revoke execute on Function_Get_Tardiness from rol_QA  
go  
  
revoke execute on Function_Get_Total_Hours_Worked from rol_QA  
go  
  
revoke execute on GetFilterClauseFromTriad from rol_QA  
go  
  
revoke select on fb_get_supervisors_by_collaborators from rol_QA  
go  
  
revoke execute on fg_get_supervisors_director_by_collaborators from rol_QA  
go  
  
  
    CREATE FUNCTION dbo.fn_diagramobjects()  
    RETURNS int  
    WITH EXECUTE AS N'dbo'  
    AS  
    BEGIN       declare @id_upgraddiagrams    int  
       declare @id_sysdiagrams          int  
       declare @id_helpdiagrams      int  
       declare @id_helpdiagramdefinition  int  
       declare @id_creatediagram  int  
       declare @id_renamediagram  int  
       declare @id_alterdiagram   int  
       declare @id_dropdiagram       int  
       declare @InstalledObjects  int  
  
       select @InstalledObjects = 0  
  
       select     @id_upgraddiagrams = object_id(N'dbo.sp_upgraddiagrams'),  
          @id_sysdiagrams = object_id(N'dbo.sysdiagrams'),  
          @id_helpdiagrams = object_id(N'dbo.sp_helpdiagrams'),  
          @id_helpdiagramdefinition = object_id(N'dbo.sp_helpdiagramdefinition'),  
          @id_creatediagram = object_id(N'dbo.sp_creatediagram'),  
          @id_renamediagram = object_id(N'dbo.sp_renamediagram'),  
          @id_alterdiagram = object_id(N'dbo.sp_alterdiagram'),  
          @id_dropdiagram = object_id(N'dbo.sp_dropdiagram')  
  
       if @id_upgraddiagrams is not null  
          select @InstalledObjects = @InstalledObjects + 1  
       if @id_sysdiagrams is not null  
          select @InstalledObjects = @InstalledObjects + 2  
       if @id_helpdiagrams is not null  
          select @InstalledObjects = @InstalledObjects + 4  
       if @id_helpdiagramdefinition is not null  
          select @InstalledObjects = @InstalledObjects + 8  
       if @id_creatediagram is not null  
          select @InstalledObjects = @InstalledObjects + 16  
       if @id_renamediagram is not null  
          select @InstalledObjects = @InstalledObjects + 32  
       if @id_alterdiagram  is not null  
          select @InstalledObjects = @InstalledObjects + 64  
       if @id_dropdiagram is not null  
          select @InstalledObjects = @InstalledObjects + 128  
  
       return @InstalledObjects  
    END  
go  
  
exec sp_addextendedproperty 'microsoft_database_tools_support', 1, 'SCHEMA', 'dbo', 'FUNCTION', 'fn_diagramobjects'  
go  
  
deny execute on fn_diagramobjects to guest  
go  
  
grant execute on fn_diagramobjects to [public]  
go  
  
revoke execute on fn_get_Real_End_Date_Requirement from rol_QA  
go  
  
revoke select on fn_get_all_supervisor_by_collaborator from rol_QA  
go  
  
revoke execute on fn_get_closing_days_by_request_date from rol_QA  
go  
  
revoke execute on fn_get_collaborators_by_supervisor from rol_QA  
go  
  
revoke execute on fn_get_days_from_year from rol_QA  
go  
  
revoke execute on fn_get_following_work_day from rol_QA  
go  
  
revoke execute on fn_get_id_supervisor_By_Collaborator from rol_QA  
go  
  
revoke execute on fn_get_marca_recomendada from rol_QA  
go  
  
revoke execute on fn_get_modalidad_colaborador from rol_QA  
go  
  
revoke execute on fn_get_rango_marca_max from rol_QA  
go  
  
revoke execute on fn_get_rango_marca_max_segun_tipo from rol_QA  
go  
  
revoke execute on fn_get_rango_marca_min from rol_QA  
go  
  
revoke execute on fn_get_rango_marca_min_segun_tipo from rol_QA  
go  
  
revoke select on fn_get_request_hours_by_Transaction from rol_QA  
go  
  
revoke execute on fn_get_weekday from rol_QA  
go  
  
revoke execute on fun_Get_Schedule_By_Collaborator from rol_QA  
go  
  
revoke execute on function_get_tolerancia from rol_QA  
go  
  
revoke execute on get_Historico_Cierre_Solicitudes from rol_QA  
go  
  
revoke execute on get_configuraciones from rol_QA  
go  
  
revoke execute on get_count_marcas from rol_QA  
go  
  
revoke execute on get_dias_laborados from rol_QA  
go  
  
revoke execute on get_fichas_sintomatologicas from rol_QA  
go  
  
revoke execute on get_footer_closure_history from rol_QA  
go  
  
revoke execute on get_marca_faltante_dashboard from rol_QA  
go  
  
revoke execute on get_marcas_pendientes from rol_QA  
go  
  
revoke execute on get_mes_español from rol_QA  
go  
  
revoke execute on get_modulos_rol from rol_QA  
go  
  
revoke execute on get_pending_marks_and_requests from rol_QA  
go  
  
revoke execute on get_tipo_marcacion_descripcion from rol_QA  
go  
  
revoke execute on sp_Approver_Get_Name_Email_UserId from rol_QA  
go  
  
revoke execute on sp_Approver_Team_Get_Name_Email_UserId from rol_QA  
go  
  
revoke execute on sp_GetApproverIdByRoleNameAndOwnerId from rol_QA  
go  
  
revoke execute on sp_GetRolesByCollaboratorId from rol_QA  
go  
  
revoke execute on sp_Get_ProyectsFilter from rol_QA  
go  
  
revoke execute on sp_Get_nId_Request_By_Approvers from rol_QA  
go  
  
revoke execute on sp_Notification_Get_Pendings_For_All_Users_By_Tenant from rol_QA  
go  
  
revoke execute on sp_Notification_Get_Pendings_In_Hour_Range_By_User from rol_QA  
go  
  
revoke execute on sp_Request_get_affecting_attendance_by_collaborator_and_date from rol_QA  
go  
  
revoke execute on sp_Task_Get_From_vw from rol_QA  
go  
  
revoke execute on sp_Task_Get_Total_Minutes_For_Non_Regular_Tasks from rol_QA  
go  
  
  
    CREATE PROCEDURE dbo.sp_alterdiagram  
    (  
       @diagramname   sysname,  
       @owner_id  int    = null,  
       @version   int,  
       @definition    varbinary(max)  
    )  
    WITH EXECUTE AS 'dbo'  
    AS  
    BEGIN       set nocount on  
       declare @theId           int  
       declare @retval       int  
       declare @IsDbo           int  
  
       declare @UIDFound     int  
       declare @DiagId          int  
       declare @ShouldChangeUID   int  
  
       if(@diagramname is null)  
       begin  
          RAISERROR ('Invalid ARG', 16, 1)  
          return -1  
       end  
  
       execute as caller;  
       select @theId = DATABASE_PRINCIPAL_ID();  
       select @IsDbo = IS_MEMBER(N'db_owner');  
       if(@owner_id is null)  
          select @owner_id = @theId;  
       revert;  
  
       select @ShouldChangeUID = 0  
       select @DiagId = diagram_id, @UIDFound = principal_id from dbo.sysdiagrams where principal_id = @owner_id and name = @diagramname  
  
       if(@DiagId IS NULL or (@IsDbo = 0 and @theId <> @UIDFound))  
       begin  
          RAISERROR ('Diagram does not exist or you do not have permission.', 16, 1);  
          return -3  
       end  
  
       if(@IsDbo <> 0)  
       begin  
          if(@UIDFound is null or USER_NAME(@UIDFound) is null) -- invalid principal_id  
          begin  
             select @ShouldChangeUID = 1 ;  
          end  
       end  
       -- update dds data  
       update dbo.sysdiagrams set definition = @definition where diagram_id = @DiagId ;  
  
       -- change owner  
       if(@ShouldChangeUID = 1)  
          update dbo.sysdiagrams set principal_id = @theId where diagram_id = @DiagId ;  
  
       -- update dds version  
       if(@version is not null)  
          update dbo.sysdiagrams set version = @version where diagram_id = @DiagId ;  
  
       return 0  
    END  
go  
  
exec sp_addextendedproperty 'microsoft_database_tools_support', 1, 'SCHEMA', 'dbo', 'PROCEDURE', 'sp_alterdiagram'  
go  
  
deny execute on sp_alterdiagram to guest  
go  
  
grant execute on sp_alterdiagram to [public]  
go  
  
revoke execute on sp_campus_get_all from rol_QA  
go  
  
revoke execute on sp_coordinator_get_detail_by_id from rol_QA  
go  
  
revoke execute on sp_coordinators_get_all from rol_QA  
go  
  
revoke execute on sp_create_asistence from rol_QA  
go  
  
  
    CREATE PROCEDURE dbo.sp_creatediagram  
    (  
       @diagramname   sysname,  
       @owner_id     int    = null,  
       @version      int,  
       @definition    varbinary(max)  
    )  
    WITH EXECUTE AS 'dbo'  
    AS  
    BEGIN       set nocount on  
       declare @theId int  
       declare @retval int  
       declare @IsDbo int  
       declare @userName sysname  
       if(@version is null or @diagramname is null)  
       begin  
          RAISERROR (N'E_INVALIDARG', 16, 1);  
          return -1  
       end  
  
       execute as caller;  
       select @theId = DATABASE_PRINCIPAL_ID();  
       select @IsDbo = IS_MEMBER(N'db_owner');  
       revert;  
  
       if @owner_id is null  
       begin          select @owner_id = @theId;  
       end  
       else       begin          if @theId <> @owner_id  
          begin  
             if @IsDbo = 0  
             begin  
                RAISERROR (N'E_INVALIDARG', 16, 1);  
                return -1  
             end  
             select @theId = @owner_id  
          end  
       end       -- next 2 line only for test, will be removed after define name unique  
       if EXISTS(select diagram_id from dbo.sysdiagrams where principal_id = @theId and name = @diagramname)  
       begin  
          RAISERROR ('The name is already used.', 16, 1);  
          return -2  
       end  
  
       insert into dbo.sysdiagrams(name, principal_id , version, definition)  
             VALUES(@diagramname, @theId, @version, @definition) ;  
  
       select @retval = @@IDENTITY  
       return @retval  
    END  
go  
  
exec sp_addextendedproperty 'microsoft_database_tools_support', 1, 'SCHEMA', 'dbo', 'PROCEDURE', 'sp_creatediagram'  
go  
  
deny execute on sp_creatediagram to guest  
go  
  
grant execute on sp_creatediagram to [public]  
go  
  
revoke execute on sp_direction_get_detail_by_nIdSede from rol_QA  
go  
  
  
    CREATE PROCEDURE dbo.sp_dropdiagram  
    (  
       @diagramname   sysname,  
       @owner_id  int    = null  
    )  
    WITH EXECUTE AS 'dbo'  
    AS  
    BEGIN       set nocount on       declare @theId           int  
       declare @IsDbo           int  
  
       declare @UIDFound     int  
       declare @DiagId          int  
  
       if(@diagramname is null)  
       begin  
          RAISERROR ('Invalid value', 16, 1);  
          return -1  
       end  
  
       EXECUTE AS CALLER;  
       select @theId = DATABASE_PRINCIPAL_ID();  
       select @IsDbo = IS_MEMBER(N'db_owner');  
       if(@owner_id is null)  
          select @owner_id = @theId;  
       REVERT;  
  
       select @DiagId = diagram_id, @UIDFound = principal_id from dbo.sysdiagrams where principal_id = @owner_id and name = @diagramname  
       if(@DiagId IS NULL or (@IsDbo = 0 and @UIDFound <> @theId))  
       begin  
          RAISERROR ('Diagram does not exist or you do not have permission.', 16, 1)  
          return -3  
       end  
  
       delete from dbo.sysdiagrams where diagram_id = @DiagId;  
  
       return 0;  
    END  
go  
  
exec sp_addextendedproperty 'microsoft_database_tools_support', 1, 'SCHEMA', 'dbo', 'PROCEDURE', 'sp_dropdiagram'  
go  
  
deny execute on sp_dropdiagram to guest  
go  
  
grant execute on sp_dropdiagram to [public]  
go  
  
revoke execute on sp_find_collaborator from rol_QA  
go  
  
revoke execute on sp_getEmail_Login from rol_QA  
go  
  
revoke execute on sp_getPlans from rol_QA  
go  
  
revoke execute on sp_get_all_Request_History from rol_QA  
go  
  
revoke execute on sp_get_asistance_in_month from rol_QA  
go  
  
revoke execute on sp_get_assistance_in_month from rol_QA  
go  
  
revoke execute on sp_get_biomtrico_Logs_by_user from rol_QA  
go  
  
revoke execute on sp_get_compensacion_recuperacion_permisos from rol_QA  
go  
  
revoke execute on sp_get_dates_condition from rol_QA  
go  
  
revoke execute on sp_get_dates_range_condition from rol_QA  
go  
  
revoke execute on sp_get_equal_condition from rol_QA  
go  
  
revoke execute on sp_get_justifications_assistence from rol_QA  
go  
  
revoke execute on sp_get_lider_director_by_nId_collaborator from rol_QA  
go  
  
revoke execute on sp_get_like_condition from rol_QA  
go  
  
revoke execute on sp_get_listado_tareas_semanales from rol_QA  
go  
  
revoke execute on sp_get_marcas_pendientes_masivas from rol_QA  
go  
  
revoke execute on sp_get_marks_fail_biometrico from rol_QA  
go  
  
revoke execute on sp_get_multiple_like_condition from rol_QA  
go  
  
revoke execute on sp_get_order_clause from rol_QA  
go  
  
revoke execute on sp_get_pagination_clause from rol_QA  
go  
  
revoke execute on sp_get_pending_requests_by_nId_collaborator from rol_QA  
go  
  
revoke execute on sp_get_positive_negative_hours_by_Collaborator from rol_QA  
go  
  
revoke execute on sp_get_recipients_ranking_tareo from rol_QA  
go  
  
revoke execute on sp_get_recipients_renew_contracts from rol_QA  
go  
  
revoke execute on sp_get_replacement_Collaborator_by_Request from rol_QA  
go  
  
revoke execute on sp_get_request_detail_assistance from rol_QA  
go  
  
revoke execute on sp_get_request_formatted_by_Id from rol_QA  
go  
  
revoke execute on sp_get_request_history from rol_QA  
go  
  
revoke execute on sp_get_request_hours_by_Transaction_old from rol_QA  
go  
  
revoke execute on sp_get_request_hours_by_day from rol_QA  
go  
  
revoke execute on sp_get_requests_for_card_by_user from rol_QA  
go  
  
revoke execute on sp_get_sSlugPermisionsByRoles from rol_QA  
go  
  
revoke execute on sp_get_saldos_vacaciones from rol_QA  
go  
  
revoke execute on sp_get_tardiness_hours_balance_by_cierre from rol_QA  
go  
  
revoke execute on sp_get_tardiness_report_by_cierre from rol_QA  
go  
  
revoke execute on sp_get_ultimas_marcas_pendientes from rol_QA  
go  
  
  
    CREATE PROCEDURE dbo.sp_helpdiagramdefinition  
    (  
       @diagramname   sysname,  
       @owner_id  int    = null  
    )  
    WITH EXECUTE AS N'dbo'  
    AS  
    BEGIN       set nocount on  
       declare @theId        int  
       declare @IsDbo        int  
       declare @DiagId       int  
       declare @UIDFound  int  
  
       if(@diagramname is null)  
       begin  
          RAISERROR (N'E_INVALIDARG', 16, 1);  
          return -1  
       end  
  
       execute as caller;  
       select @theId = DATABASE_PRINCIPAL_ID();  
       select @IsDbo = IS_MEMBER(N'db_owner');  
       if(@owner_id is null)  
          select @owner_id = @theId;  
       revert;  
  
       select @DiagId = diagram_id, @UIDFound = principal_id from dbo.sysdiagrams where principal_id = @owner_id and name = @diagramname;  
       if(@DiagId IS NULL or (@IsDbo = 0 and @UIDFound <> @theId ))  
       begin  
          RAISERROR ('Diagram does not exist or you do not have permission.', 16, 1);  
          return -3  
       end  
  
       select version, definition FROM dbo.sysdiagrams where diagram_id = @DiagId ;  
       return 0  
    END  
go  
  
exec sp_addextendedproperty 'microsoft_database_tools_support', 1, 'SCHEMA', 'dbo', 'PROCEDURE',  
     'sp_helpdiagramdefinition'  
go  
  
deny execute on sp_helpdiagramdefinition to guest  
go  
  
grant execute on sp_helpdiagramdefinition to [public]  
go  
  
  
    CREATE PROCEDURE dbo.sp_helpdiagrams  
    (  
       @diagramname sysname = NULL,  
       @owner_id int = NULL  
    )  
    WITH EXECUTE AS N'dbo'  
    AS  
    BEGIN       DECLARE @user sysname  
       DECLARE @dboLogin bit  
       EXECUTE AS CALLER;  
          SET @user = USER_NAME();  
          SET @dboLogin = CONVERT(bit,IS_MEMBER('db_owner'));  
       REVERT;  
       SELECT  
          [Database] = DB_NAME(),  
          [Name] = name,  
          [ID] = diagram_id,  
          [Owner] = USER_NAME(principal_id),  
          [OwnerID] = principal_id  
       FROM  
          sysdiagrams  
       WHERE  
          (@dboLogin = 1 OR USER_NAME(principal_id) = @user) AND  
          (@diagramname IS NULL OR name = @diagramname) AND  
          (@owner_id IS NULL OR principal_id = @owner_id)  
       ORDER BY  
          4, 5, 1  
    END  
go  
  
exec sp_addextendedproperty 'microsoft_database_tools_support', 1, 'SCHEMA', 'dbo', 'PROCEDURE', 'sp_helpdiagrams'  
go  
  
deny execute on sp_helpdiagrams to guest  
go  
  
grant execute on sp_helpdiagrams to [public]  
go  
  
revoke execute on sp_questionsAndAnswers_getById from rol_QA  
go  
  
  
    CREATE PROCEDURE dbo.sp_renamediagram  
    (  
       @diagramname      sysname,  
       @owner_id     int    = null,  
       @new_diagramname   sysname  
  
    )  
    WITH EXECUTE AS 'dbo'  
    AS  
    BEGIN       set nocount on       declare @theId           int  
       declare @IsDbo           int  
  
       declare @UIDFound     int  
       declare @DiagId          int  
       declare @DiagIdTarg       int  
       declare @u_name          sysname  
       if((@diagramname is null) or (@new_diagramname is null))  
       begin  
          RAISERROR ('Invalid value', 16, 1);  
          return -1  
       end  
  
       EXECUTE AS CALLER;  
       select @theId = DATABASE_PRINCIPAL_ID();  
       select @IsDbo = IS_MEMBER(N'db_owner');  
       if(@owner_id is null)  
          select @owner_id = @theId;  
       REVERT;  
  
       select @u_name = USER_NAME(@owner_id)  
  
       select @DiagId = diagram_id, @UIDFound = principal_id from dbo.sysdiagrams where principal_id = @owner_id and name = @diagramname  
       if(@DiagId IS NULL or (@IsDbo = 0 and @UIDFound <> @theId))  
       begin  
          RAISERROR ('Diagram does not exist or you do not have permission.', 16, 1)  
          return -3  
       end  
  
       -- if((@u_name is not null) and (@new_diagramname = @diagramname)) -- nothing will change  
       -- return 0;  
       if(@u_name is null)  
          select @DiagIdTarg = diagram_id from dbo.sysdiagrams where principal_id = @theId and name = @new_diagramname  
       else  
          select @DiagIdTarg = diagram_id from dbo.sysdiagrams where principal_id = @owner_id and name = @new_diagramname  
  
       if((@DiagIdTarg is not null) and  @DiagId <> @DiagIdTarg)  
       begin  
          RAISERROR ('The name is already used.', 16, 1);  
          return -2  
       end  
  
       if(@u_name is null)  
          update dbo.sysdiagrams set [name] = @new_diagramname, principal_id = @theId where diagram_id = @DiagId  
       else  
          update dbo.sysdiagrams set [name] = @new_diagramname where diagram_id = @DiagId  
       return 0  
    END  
go  
  
exec sp_addextendedproperty 'microsoft_database_tools_support', 1, 'SCHEMA', 'dbo', 'PROCEDURE', 'sp_renamediagram'  
go  
  
deny execute on sp_renamediagram to guest  
go  
  
grant execute on sp_renamediagram to [public]  
go  
  
revoke execute on sp_smart_GetClosingDateInfo from rol_QA  
go  
  
revoke execute on sp_smart_Position_GetAll from rol_QA  
go  
  
revoke execute on sp_smart_Positon_get_detail from rol_QA  
go  
  
revoke execute on sp_smart_Responsabilities_get_detail from rol_QA  
go  
  
revoke execute on sp_smart_campus_get_detail from rol_QA  
go  
  
revoke execute on sp_smart_direction_get_by_id_persona from rol_QA  
go  
  
CREATE   PROCEDURE [dbo].[sp_smart_evaluation_cancellation](  
    @nId_Evaluation INT  
)  
AS  
BEGIN  
    DELETE Evaluations WHERE nId_Evaluation = @nId_Evaluation  
END  
go  
  
alter   PROCEDURE [dbo].[sp_smart_evaluation_create](  
    @UserCreator INT,  
    @sTitle NVARCHAR(MAX),  
    @sDescription NVARCHAR(MAX),  
    @dInit_Date DATETIME,  
    @dEnd_Date DATETIME,  
    @sForms NVARCHAR(MAX)  
)  
AS  
BEGIN  
    --SET NOCOUNT ON;  
    BEGIN TRY  
        BEGIN TRANSACTION;  
          DECLARE @FormTypeTemplate AS TABLE (nId_FormType INT, nId_Template INT);  
  
          INSERT INTO @FormTypeTemplate  
          SELECT  
             f.nId_FormType,  
             f.nId_Template  
          FROM OPENJSON(@sForms)  
          WITH (  
             nId_FormType INT,  
             nId_Template INT  
          ) as f  
  
          IF EXISTS(  
             SELECT 1  
             FROM @FormTypeTemplate ftt  
             LEFT JOIN Form_Template ft on ftt.nId_Template = ft.nId_Form_Template  
             WHERE ft.nId_Form_Template IS NULL  
          )  
          BEGIN  
              RAISERROR('No existe una de las plantillas', 16, 1);  
          END  
  
          IF EXISTS(  
             SELECT 1  
             FROM @FormTypeTemplate ftt  
             LEFT JOIN Form_Type ft on ftt.nId_FormType = ft.nId_Form_Type  
             WHERE ft.nId_Form_Type IS NULL OR ft.nId_Form_Type IN (  
                   SELECT  
                      form.nId_FormType_Padre  
                   FROM Form_Type form  
                   WHERE form.nId_FormType_Padre IS NOT NULL  
                )  
          )  
          BEGIN  
              RAISERROR('No existe uno de los tipos de evaluaciones', 16, 1);  
          END  
  
  
          --INSERT EVALUATION  
          DECLARE @nId_Evaluation INT;  
          INSERT INTO Evaluations (sName, nState, sDescription, dInit_Date, dEnd_Date, nUser_Creator, dDateTime_Creator)  
          VALUES (@sTitle, 1, @sDescription, @dInit_Date, @dEnd_Date, @UserCreator, DATEADD(HOUR, -5, GETDATE()));  
  
          SET @nId_Evaluation = SCOPE_IDENTITY();  
  
          --INSERT FORMS  
          DECLARE @FormTable AS TABLE (nId_Form INT ,nId_Form_Type INT, nId_Form_SubType INT);  
          INSERT INTO Forms (sName, sDescription, nId_Evaluation, nUser_Creator, dDateTime_Creator, nId_Form_Type, nId_Form_SubType, nId_Form_Template)  
          OUTPUT inserted.nId_Form, inserted.nId_Form_Type, inserted.nId_Form_SubType INTO @FormTable  
          SELECT  
             ft.sName,  
             ft.sName as sDescription,  
             @nId_Evaluation as nId_Evaluation,  
             @UserCreator as nUser_Creator,  
             DATEADD(HOUR, -5, GETDATE()) as dDateTime_Creator,  
             (CASE  
                WHEN ft.nId_FormType_Padre IS NULL THEN ft.nId_Form_Type  
                ELSE ft.nId_FormType_Padre  
             END) as nId_Form_Type,  
             (CASE  
                WHEN ft.nId_FormType_Padre IS NULL THEN NULL  
                ELSE ft.nId_Form_type  
             END) as nId_Form_SubType,  
             t.nId_Template as nId_Form_Template  
          FROM @FormTypeTemplate t  
          JOIN Form_Type ft on t.nId_FormType = ft.nId_Form_Type  
  
          --INSERT QUESTIONS  
          CREATE TABLE #QuestionsMap (  
              nId_Question_Template INT,  
              nId_Question INT  
          );  
  
          INSERT INTO Questions (nId_Form, sText, nId_Type_Question, bRequired, sPlaceHolder, nId_Competence, nUser_Creator, dDateTime_Creator, nId_Question_Template)  
          OUTPUT inserted.nId_Question_Template, inserted.nId_Question INTO #QuestionsMap  
          SELECT  
             formt.nId_Form,  
             qt.sText,  
             qt.nId_Type_Question,  
             qt.bRequired,  
             qt.sPlaceHolder,  
             qt.nId_Competence,  
             @UserCreator,  
             DATEADD(HOUR, -5, GETDATE()) as dDateTime_Creator,  
             qt.nId_Question  
          FROM Form_Template ft  
          JOIN Questions_Template qt ON ft.nId_Form_Template = qt.nId_Form_Template  
          JOIN @FormTypeTemplate ftt ON ft.nId_Form_Template = ftt.nId_Template  
          JOIN @FormTable formt ON (  
           (formt.nId_Form_SubType IS NULL AND ftt.nId_FormType = formt.nId_Form_Type)  
           OR  
           (formt.nId_Form_SubType IS NOT NULL AND ftt.nId_FormType = formt.nId_Form_SubType)  
          ) ORDER BY formt.nId_Form  
  
          --INSERT data lists from templates  
          INSERT INTO data_list (  
              nId_Question,  
              sText,  
              nValue,  
              nUser_Creator,  
              dDatetime_Creator  
          )  
          SELECT  
              qm.nId_Question,  
              dlt.sText,  
              dlt.nValue,  
              @UserCreator,  
              DATEADD(HOUR, -5, GETDATE())  
          FROM dataList_Template dlt  
          JOIN #QuestionsMap qm ON dlt.nId_Question = qm.nId_Question_Template;  
  
          --INSERT EVALUATORS  
  
          DECLARE @FormTypes NVARCHAR(100);  
          SELECT  
             @FormTypes = STRING_AGG(t.nId_FormType, ',')  
          FROM @FormTypeTemplate t  
  
          CREATE TABLE #Evaluators (  
             nId_Evaluator INT,  
             sEvaluator_Name VARCHAR(MAX),  
             nId_Assessed INT,  
             sAssessed_Name VARCHAR(MAX),  
             nId_FormType INT  
          )  
  
          INSERT INTO #Evaluators  
          EXEC sp_smart_evaluation_get_evaluators_assessed_template NULL, @FormTypes;  
  
          INSERT INTO Evaluators  
          SELECT  
             e.nId_Evaluator,  
             e.nId_Assessed,  
             ft.nId_Form  
          FROM #Evaluators e  
          JOIN @FormTable ft ON  
          (  
           (ft.nId_Form_SubType IS NULL AND e.nId_FormType = ft.nId_Form_Type)  
           OR  
           (ft.nId_Form_SubType IS NOT NULL AND e.nId_FormType = ft.nId_Form_SubType)  
          )  
  
          SELECT  
             @nId_Evaluation AS nId_Evaluation  
  
       COMMIT TRANSACTION;  
  
    END TRY  
    BEGIN CATCH        IF @@TRANCOUNT > 0  
            ROLLBACK TRANSACTION;  
  
        THROW;  
    END CATCH  
  
END  
go  
  
alter     PROCEDURE [dbo].[sp_smart_evaluation_edit_evaluators_assessed](  
    @nId_Evaluation NVARCHAR(MAX),  
    @nId_Evaluator INT,  
    @AssessedList AssessedFormTypeList READONLY  
)  
AS  
BEGIN  
    BEGIN TRY        BEGIN TRANSACTION;  
  
          --Comprobar si es supervisor  
          DECLARE @IsSupervisor BIT;  
  
          SELECT @IsSupervisor =  
                CASE  
                   WHEN EXISTS (  
                         SELECT 1  
                         FROM @AssessedList al  
                         WHERE al.nId_FormType = 3 --tipo subordinados  
                      ) THEN 1  
                   ELSE 0  
                END  
  
          --BULK DELETE  
          DELETE e  
          FROM Evaluators e  
          JOIN Forms f ON e.nId_Form = f.nId_Form  
          WHERE e.nId_Evaluator = @nId_Evaluator AND f.nId_Evaluation = @nId_Evaluation  
  
          --INSERT si el formulario no tiene subtipos  
  
          INSERT INTO Evaluators  
          SELECT  
             @nId_Evaluator AS nId_Evaluator,  
             al.nId_Assessed,  
             f.nId_Form  
          FROM @AssessedList al  
          JOIN Forms f ON al.nId_FormType = f.nId_Form_Type  
          WHERE f.nId_Form_SubType IS NULL AND f.nId_Evaluation = @nId_Evaluation  
  
          --INSERTS EN CASO DE PARES  
          IF EXISTS (  
                SELECT 1 FROM @AssessedList WHERE nId_FormType = 6  
             )  
          BEGIN  
  
             INSERT INTO Evaluators  
             SELECT  
                @nId_Evaluator AS nId_Evaluator,  
                al.nId_Assessed,  
                f.nId_Form  
             FROM @AssessedList al  
             JOIN Forms f ON al.nId_FormType = f.nId_Form_Type  
             WHERE f.nId_Form_SubType IS NOT NULL  
                AND f.nId_Evaluation = @nId_Evaluation  
                AND f.nId_Form_Type = 6 --tipo pares  
                AND f.nId_Form_SubType =  
                   (CASE  
                      WHEN @IsSupervisor = 1 THEN 8 --Lideres  
                      ELSE 7 --Colaborador  
                   END)  
          END  
  
          --Caso Subordinados  
  
          IF EXISTS (  
                SELECT 1 FROM @AssessedList WHERE nId_FormType = 3  
             )  
          BEGIN  
  
             DECLARE @nNewId_Form INT  
  
             --Obtener evaluado y a que subtipo de evaluacion pertenece  
             SELECT  
                al.nId_Assessed,  
                CASE  
                   WHEN f.nId_Form_Type IS NULL THEN 4  
                   ELSE 5  
                END as nId_Form_SubType  
             INTO #Subordinates_supervisor  
             FROM @AssessedList al  
             LEFT JOIN Evaluators e ON al.nId_Assessed = e.nId_Evaluator  
                AND e.nId_Form IN (  
                      SELECT  
                         form.nId_Form  
                      FROM forms form  
                      WHERE form.nId_Evaluation = @nId_Evaluation  
                      AND form.nId_Form_Type = 3  
                   )  
             LEFT JOIN Forms f on e.nId_Form = f.nId_Form  
             WHERE al.nId_FormType = 3  
             GROUP BY al.nId_Assessed, f.nId_Form_Type  
  
             --Insert de evaluados Subordinados  
             INSERT INTO Evaluators  
             SELECT  
                @nId_Evaluator AS nId_Evaluator,  
                al.nId_Assessed,  
                f.nId_Form  
             FROM #Subordinates_supervisor al  
             JOIN Forms f ON al.nId_Form_SubType = f.nId_Form_SubType  
             WHERE f.nId_Evaluation = @nId_Evaluation  
  
          END  
          --update del evaluador donde es evaluado  
          SELECT  
             f.nId_Form,  
             f.nId_Form_SubType  
          INTO #Forms  
          FROM Forms f  
          WHERE f.nId_Evaluation = 21  
          AND f.nId_Form_Type = 3  
  
          SELECT @nNewId_Form = f2.nId_Form  
          FROM #Forms f2  
          WHERE f2.nId_Form_SubType =  
             CASE  
                 WHEN @IsSupervisor = 1 THEN 5  
                 ELSE 4  
             END;  
  
          UPDATE e  
          SET e.nId_Form = @nNewId_Form  
          FROM Evaluators e  
          JOIN Forms f ON e.nId_Form = f.nId_Form  
          WHERE  
             e.nId_Assessed = @nId_Evaluator  
             AND f.nId_Evaluation = @nId_Evaluation  
             AND f.nId_Form_Type = 3  
  
       COMMIT TRANSACTION;  
    END TRY  
    BEGIN CATCH        IF @@TRANCOUNT > 0  
            ROLLBACK TRANSACTION;  
  
        THROW;  
    END CATCH  
END  
go  
  
alter   PROCEDURE [dbo].[sp_smart_evaluation_evaluator_assessed_form_questions]  
(  
    @nId_Evaluation INT,  
    @nId_Evaluator VARCHAR(MAX) = NULL,  
    @nId_Assessed VARCHAR(MAX) = NULL,  
    @pnum INT = NULL, --Número de Páginas  
    @size INT = NULL --Cantidad de Registros por página  
)  
AS  
BEGIN  
    DECLARE @script AS nvarchar(max)  
    DECLARE @whereClause AS nvarchar(max) = ''  
    DECLARE @paginationClause AS nvarchar(max)  
    DECLARE @conditions AS TABLE (condition nvarchar(max))  
    DECLARE @conditionOUT AS nvarchar(max);  
  
    IF @nId_Evaluator IS NOT NULL  
    BEGIN       INSERT INTO @conditions VALUES (N'e.nId_Evaluator IN (' + @nId_Evaluator + ')');  
    END  
  
    IF @nId_Assessed IS NOT NULL  
    BEGIN       INSERT INTO @conditions VALUES (N'e.nId_Assessed IN (' + @nId_Assessed + ')');  
    END  
  
    --ADD CONDITIONS  
    DECLARE @count INT  
  
    SELECT @count = COUNT(*) FROM @conditions  
  
    SET @whereClause = ''  
  
    WHILE @count > 0  
    BEGIN  
       DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)  
  
       SET @whereClause = concat (@whereClause,'and ', @condition_temp)  
  
       DELETE TOP (1) FROM @conditions  
       SELECT @count = COUNT(*) FROM @conditions  
    END  
  
  
    --PAGINATION  
    IF @pnum IS NULL AND @size IS NULL  
    BEGIN        SET @paginationClause = ''  
    END  
    ELSE    BEGIN        EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT  
    END    SET @script = N'  
       SELECT *       FROM v_list_evaluator_assessed_form_questions e       WHERE e.nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation) + @whereClause +'  
       ORDER BY e.nId_Question       ' + @paginationClause  
    EXECUTE sp_executesql @script  
END  
go  
  
alter PROCEDURE sp_smart_evaluation_getFormTypeAndTemplate  
AS  
BEGIN  
    SET NOCOUNT ON;  
  
    SELECT  
        nId_Form_Type AS nId,  
        sName,  
        sIcon,  
        sDescription  
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
        nId_Form_Template AS nId,        sName,        sDescription    FROM Form_Template;*/END  
go  
  
CREATE   PROCEDURE [dbo].[sp_smart_evaluation_get_assessed_by_evaluator]  
(  
    @nId_Evaluation INT,  
    @nId_Evaluator INT  
)  
AS  
BEGIN  
    SELECT       c.nId_Colaborador AS nId_Evaluador,  
       p.sPersona_Nombre AS sNombre_Evaluador,  
       c2.nId_Colaborador as nId_Evaluado,  
       p2.sPersona_Nombre as sNombre_Evaluado,  
       f.nId_Form_Type as nId_Tipo_Evaluacion,  
       ft.sName as sNombre_evaluacion  
    FROM Evaluators e  
    JOIN Colaboradores c ON e.nId_Evaluator = c.nId_Colaborador  
    JOIN Personas p ON c.nId_Persona = p.nId_Persona  
    JOIN Colaboradores c2 ON e.nId_Assessed = c2.nId_Colaborador  
    JOIN Personas p2 ON c2.nId_Persona = p2.nId_Persona  
    JOIN Forms f ON e.nId_Form = f.nId_Form  
    JOIN Form_Type ft ON ft.nId_Form_Type = f.nId_Form_Type  
    WHERE  
       f.nId_Evaluation = @nId_Evaluation  
       AND c.nId_Colaborador = @nId_Evaluator  
    ORDER BY ft.nId_Form_Type  
END  
go  
  
alter   PROCEDURE sp_smart_evaluation_get_create_resume  
(  
    @nId_Evaluation INT  
)  
AS  
BEGIN  
    SELECT *  
    FROM v_list_evaluation_create_resume l  
    WHERE l.nId_Evaluation = @nId_Evaluation  
END  
go  
  
alter   PROCEDURE [dbo].[sp_smart_evaluation_get_evaluation_resume]  
(  
    @sEvaluation_Name VARCHAR(MAX) = NULL,  
    @dStartDate VARCHAR(MAX) = NULL,  
    @dEndDate VARCHAR(MAX) = NULL,  
    @sFilterOne VARCHAR(MAX) = NULL, --FILTRO POR ESTADO  
    @pnum INT = NULL, --Número de Páginas  
    @size INT = NULL --Cantidad de Registros por página  
)  
AS  
BEGIN  
    DECLARE @script AS nvarchar(max)  
    DECLARE @whereClause AS nvarchar(max) = ''  
    DECLARE @paginationClause AS nvarchar(max)  
    DECLARE @conditions AS TABLE (condition nvarchar(max))  
    DECLARE @conditionOUT AS nvarchar(max);  
  
    IF @sEvaluation_Name IS NOT NULL  
    BEGIN       EXEC sp_get_like_condition 'sEvaluation_Name', @sEvaluation_Name, @conditionOUT OUTPUT  
        INSERT INTO @conditions VALUES (@conditionOUT);  
    END  
  
    IF @sFilterOne IS NOT NULL  
    BEGIN       INSERT INTO @conditions VALUES (N'nState IN (' + @sFilterOne + ')');  
    END  
  
    IF @dStartDate IS NOT NULL AND @dEndDate IS NOT NULL  
    BEGIN       SET @dStartDate = CONCAT(@dStartDate, ' 00:00:00.000')  
       SET @dEndDate = CONCAT(@dEndDate, ' 23:59:59.999')  
  
       SELECT @conditionOUT = CONCAT('dDateTime_Creator',' BETWEEN ''', CONVERT(DATETIME2(3), @dStartDate),''' AND ''', CONVERT(DATETIME2(3), @dEndDate), '''');  
        INSERT INTO @conditions VALUES (@conditionOUT)  
    END  
    --Agregar condicionales  
    DECLARE @count INT  
  
    SELECT @count = COUNT(*) FROM @conditions  
    IF @count > 0  
       BEGIN  
          SET @whereClause = N'WHERE '  
       END  
  
    WHILE @count > 0  
       BEGIN  
          DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)  
          IF @count = 1  
             BEGIN  
                SET @whereClause = concat (@whereClause,@condition_temp)  
             END  
          ELSE             BEGIN                SET @whereClause = concat (@whereClause,@condition_temp,' and ')  
             END  
          DELETE TOP (1) FROM @conditions  
          SELECT @count = COUNT(*) FROM @conditions  
       END  
  
    --PAGINATION  
    IF @pnum IS NULL AND @size IS NULL  
    BEGIN        SET @paginationClause = ''  
    END  
    ELSE    BEGIN        EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT  
    END  
    --Principal Query  
    SET @script = N'  
       SELECT          *,          COUNT(*) OVER() AS nTotal_Registers       FROM v_list_evaluation_resume       '       + @whereClause +  
       '  
       ORDER BY nId_Evaluation ' + @paginationClause  
  
    EXECUTE sp_executesql @script  
  
END  
go  
  
alter   PROCEDURE [dbo].[sp_smart_evaluation_get_evaluators_assessed]  
(  
    @nId_Evaluation INT,  
    @sFilterOne VARCHAR(MAX) = NULL,--FILTRO EQUIPOS  
    @sFilterTwo VARCHAR(MAX) = NULL,-- FILTRO POR COLABORADOR  
    @sFilterThree VARCHAR(MAX) = NULL, --FILTRO POR ESTADO DE CUESTIONARIO (RESPONDIDO, POR RESPONDER)  
    @pnum INT = NULL, --Número de Páginas  
    @size INT = NULL, --Cantidad de Registros por página  
    @sEvaluator_Name VARCHAR(MAX) = NULL  
)  
AS  
BEGIN  
    DECLARE @script AS nvarchar(max)  
    DECLARE @whereClause AS nvarchar(max)  
    DECLARE @paginationClause AS nvarchar(max)  
    DECLARE @conditions AS TABLE (condition nvarchar(max))  
    DECLARE @conditionOUT AS nvarchar(max);  
    DECLARE @whereClauseState AS NVARCHAR(MAX) = '';  
    --TEAM FILTER  
    IF @sFilterOne IS NOT NULL  
    BEGIN       DECLARE @CollaboratorsListTeam VARCHAR(MAX);  
       SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)  
       INSERT INTO @conditions VALUES(N'e.nId_Evaluator in('+@CollaboratorsListTeam+')')  
    END  
  
    IF @sFilterTwo IS NOT NULL  
    BEGIN       INSERT INTO @conditions VALUES(N'e.nId_Evaluator = '+@sFilterTwo)  
    END  
  
    IF @sEvaluator_Name IS NOT NULL  
    BEGIN       EXEC sp_get_like_condition 'p.sPersona_Nombre', @sEvaluator_Name, @conditionOUT OUTPUT  
        INSERT INTO @conditions VALUES (@conditionOUT);  
    END  
  
    IF @sFilterThree IS NOT NULL  
    BEGIN       SET @whereClauseState = 'AND lea.nState IN('+@sFilterThree+')'  
    END  
  
    --ADD CONDITIONS  
    DECLARE @count INT  
  
    SELECT @count = COUNT(*) FROM @conditions  
  
    SET @whereClause = ''  
  
    WHILE @count > 0  
    BEGIN  
       DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)  
  
       SET @whereClause = concat (@whereClause,'and ', @condition_temp)  
  
       DELETE TOP (1) FROM @conditions  
       SELECT @count = COUNT(*) FROM @conditions  
    END  
  
    --PAGINATION  
    IF @pnum IS NULL AND @size IS NULL  
    BEGIN        SET @paginationClause = ''  
    END  
    ELSE    BEGIN        EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT  
    END  
    --Principal query  
    SET @script = N'  
       SELECT          c.nId_Colaborador AS nId_Evaluator,          p.sPersona_Nombre AS sName_Evaluator,          f.nId_Evaluation,          e1.sName as sEvaluation_Name,          e1.sDescription,          e1.dInit_Date,          e1.dEnd_Date,          CASE             WHEN e1.nState = 1 THEN 1             WHEN GETDATE() BETWEEN e1.dInit_Date AND e1.dEnd_Date THEN 2             WHEN GETDATE() < e1.dInit_Date THEN 4             WHEN GETDATE() > e1.dEnd_Date THEN 3          END AS nState,          (             SELECT                lea.nId_Assessed,                lea.sAssessed_Name,                lea.nId_Form_Type,                lea.sFormType_Name,                lea.sIcon,                lea.nState             FROM v_list_evaluators_assessed lea             WHERE lea.nId_Evaluator = e.nId_Evaluator AND lea.nId_Evaluation = f.nId_Evaluation '+@whereClauseState+'  
             ORDER BY lea.nId_Form_Type             FOR JSON PATH          ) AS sEvaluators,          COUNT(*) OVER() AS nTotal_Registers       FROM Evaluators e       JOIN Colaboradores c ON e.nId_Evaluator = c.nId_Colaborador       JOIN Personas p ON c.nId_Persona = p.nId_Persona       JOIN Forms f on e.nId_Form = f.nId_Form       JOIN Evaluations e1 ON f.nId_Evaluation = e1.nId_Evaluation       WHERE f.nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation) + @whereClause +'  
       GROUP BY c.nId_Colaborador, p.sPersona_Nombre, e.nId_Evaluator, f.nId_Evaluation, e1.sName, e1.dInit_Date, e1.dEnd_Date, e1.sDescription, e1.nState       ORDER BY e.nId_Evaluator ' +  
       @paginationClause  
  
    EXECUTE sp_executesql @script  
  
END  
go  
  
alter PROCEDURE [dbo].[sp_smart_evaluation_get_evaluators_assessed_template](  
@nId_Collaborator NVARCHAR(MAX) = NULL,  
@nTipo_Evaluation NVARCHAR(MAX) --IDs de la tabla Form_Type  
)  
AS  
BEGIN  
    DECLARE @aColaboradores TABLE (nId_Colaborador INT);  
  
    INSERT INTO @aColaboradores (nId_Colaborador)  
    SELECT CAST(value AS INT) AS nId_Colaborador  
    FROM STRING_SPLIT(@nId_Collaborator, ',');  
  
    DECLARE @aFormType TABLE (nId_FormType INT);  
    INSERT INTO @aFormType (nId_FormType)  
    SELECT CAST(value AS INT) AS nId_FormType  
    FROM STRING_SPLIT(@nTipo_Evaluation, ',');  
  
    CREATE TABLE #Evaluators (  
        nId_Evaluator INT,  
        sEvaluator_Name NVARCHAR(100),  
        nId_Assessed INT,  
        sAssessed_Name NVARCHAR(100),  
       nId_FormType INT  
    );  
  
    --Create tabla lideres a equipos  
    SELECT  
       c.nId_Colaborador as nId_Evaluator,  
       pc.sPersona_Nombre as Evaluator_Name,  
       cs.nId_Colaborador as nId_Assessed,  
       pcs.sPersona_Nombre as Assessed_Name  
    INTO #Lideres_Equipos  
    FROM Colaboradores c  
    JOIN Personas pc on c.nId_Persona = pc.nId_Persona  
    JOIN colaboradores_supervisor cs  
       ON c.nId_Colaborador = cs.nId_Supervisor  
       AND cs.nEstado = 1  
       AND cs.bPrincipal = 1  
    JOIN Colaboradores ccs on cs.nId_Colaborador = ccs.nId_Colaborador  
    JOIN Personas pcs on ccs.nId_Persona = pcs.nId_Persona  
    WHERE c.nEstado_Colaborador = 1  
         AND ccs.nEstado_Colaborador = 1  
  
    ---------Supervisores  
   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 2)  
      OR NOT EXISTS(SELECT 1 FROM @aFormType)  
    BEGIN  
        INSERT INTO #Evaluators (nId_Evaluator, sEvaluator_Name, nId_Assessed, sAssessed_Name, nId_FormType)  
        SELECT  
            c.nId_Colaborador AS nId_Evaluator,  
            p1.sPersona_Nombre AS Evaluator_Name,  
            cs.nId_Supervisor AS nId_Assessed,  
            p2.sPersona_Nombre AS Assessed_Name,  
          2  
        FROM colaboradores_supervisor cs  
        INNER JOIN Colaboradores c ON cs.nId_Colaborador = c.nId_Colaborador  
        INNER JOIN Personas p1 ON c.nId_Persona = p1.nId_Persona  
        INNER JOIN Colaboradores s ON cs.nId_Supervisor = s.nId_Colaborador  
        INNER JOIN Personas p2 ON s.nId_Persona = p2.nId_Persona  
        WHERE (  
              NOT EXISTS (SELECT 1 FROM @aColaboradores)  
             OR c.nId_Colaborador IN (SELECT nId_Colaborador FROM @aColaboradores)  
            )  
        AND cs.nEstado = 1  
        AND c.nEstado_Colaborador = 1  
       AND cs.bPrincipal = 1  
        AND s.nEstado_Colaborador = 1;  
    END  
  
    ------------Subordinados Colaborador  
   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 4)  
      OR NOT EXISTS(SELECT 1 FROM @aFormType)  
   BEGIN  
       INSERT INTO #Evaluators  
       SELECT  
          le.nId_Evaluator,  
          le.Evaluator_Name,  
          le.nId_Assessed,  
          le.Assessed_Name,  
          4  
       FROM #Lideres_Equipos le  
       WHERE le.nId_Assessed NOT IN (  
             SELECT  
                l.nId_Evaluator  
             FROM #Lideres_Equipos l  
          )  
   END  
  
   --------------Subordinados Lideres  
   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 5)  
      OR NOT EXISTS(SELECT 1 FROM @aFormType)  
   BEGIN  
       INSERT INTO #Evaluators  
       SELECT  
          le.nId_Evaluator,  
          le.Evaluator_Name,  
          le.nId_Assessed,  
          le.Assessed_Name,  
          5  
       FROM #Lideres_Equipos le  
       WHERE  
          (  
              NOT EXISTS (SELECT 1 FROM @aColaboradores)  
             OR le.nId_Evaluator IN (SELECT nId_Colaborador FROM @aColaboradores)  
          )  
          AND le.nId_Assessed IN (  
             SELECT  
                l.nId_Evaluator  
             FROM #Lideres_Equipos l  
          )  
   END  
  
   --Pares Colaboradores  
  
   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 7)  
      OR NOT EXISTS(SELECT 1 FROM @aFormType)  
   BEGIN  
       SELECT            c.nId_Colaborador AS nId_Evaluator,  
            p1.sPersona_Nombre AS Evaluator_Name,  
            cs.nId_Supervisor AS nId_Assessed,  
            p2.sPersona_Nombre AS Assessed_Name  
       INTO #Colaborador_Lideres  
        FROM colaboradores_supervisor cs  
        INNER JOIN Colaboradores c ON cs.nId_Colaborador = c.nId_Colaborador  
        INNER JOIN Personas p1 ON c.nId_Persona = p1.nId_Persona  
        INNER JOIN Colaboradores s ON cs.nId_Supervisor = s.nId_Colaborador  
        INNER JOIN Personas p2 ON s.nId_Persona = p2.nId_Persona  
        WHERE (  
              NOT EXISTS (SELECT 1 FROM @aColaboradores)  
             OR c.nId_Colaborador IN (SELECT nId_Colaborador FROM @aColaboradores)  
            )  
        AND cs.nEstado = 1  
        AND c.nEstado_Colaborador = 1  
       AND cs.bPrincipal = 1  
        AND s.nEstado_Colaborador = 1;  
       ------------------  
       INSERT INTO #Evaluators  
       SELECT  
          cl.nId_Evaluator,  
          cl.Evaluator_Name,  
          le.nId_Assessed,  
          le.Assessed_Name,  
          7 as nId_FormType  
       FROM #Colaborador_Lideres cl  
       JOIN #Lideres_Equipos le ON cl.nId_Assessed = le.nId_Evaluator  
       WHERE  
          cl.nId_Evaluator != le.nId_Assessed  
          and cl.nId_Evaluator not in (select nId_Evaluator  
          from #Lideres_Equipos)  
  
   END  
  
   ---------Pares Lideres  
   IF EXISTS(SELECT 1 FROM @aFormType WHERE nId_FormType = 8)  
      OR NOT EXISTS(SELECT 1 FROM @aFormType)  
    BEGIN  
        DECLARE @EsLider BIT = 0;  
  
        IF @nId_Collaborator IS NOT NULL  
        BEGIN            SELECT @EsLider = 1  
            FROM colaboradores_supervisor cs  
            WHERE cs.nId_Supervisor = @nId_Collaborator  
            AND cs.nEstado = 1;  
        END  
  
        INSERT INTO #Evaluators  
        SELECT  
            Evaluador.nId_Colaborador AS nId_Evaluator,  
            p1.sPersona_Nombre AS sEvaluator_Name,  
            Evaluado.nId_Colaborador AS nId_Assessed,  
            p2.sPersona_Nombre AS sAssessed_Name,  
          8 as nId_FormType  
        FROM  
            (SELECT DISTINCT cs.nId_Supervisor AS nId_Colaborador  
             FROM colaboradores_supervisor cs  
             WHERE cs.nId_Supervisor IS NOT NULL  
             AND cs.nEstado = 1  
             AND EXISTS (  
                 SELECT 1  
                 FROM colaboradores_supervisor cs2  
                 WHERE cs2.nId_Colaborador = cs.nId_Supervisor  
                 AND cs2.nId_Supervisor IS NOT NULL  
             )  
             AND (@nId_Collaborator IS NULL OR @EsLider = 0 OR cs.nId_Supervisor = @nId_Collaborator)) Evaluador  
        CROSS JOIN  
            (SELECT DISTINCT cs.nId_Supervisor AS nId_Colaborador  
             FROM colaboradores_supervisor cs  
             WHERE cs.nId_Supervisor IS NOT NULL  
             AND cs.nEstado = 1  
             AND EXISTS (  
                 SELECT 1  
                 FROM colaboradores_supervisor cs2  
                 WHERE cs2.nId_Colaborador = cs.nId_Supervisor  
                 AND cs2.nId_Supervisor IS NOT NULL  
             )) Evaluado  
        INNER JOIN Colaboradores c1 ON Evaluador.nId_Colaborador = c1.nId_Colaborador  
        INNER JOIN Personas p1 ON c1.nId_Persona = p1.nId_Persona  
        INNER JOIN Colaboradores c2 ON Evaluado.nId_Colaborador = c2.nId_Colaborador  
        INNER JOIN Personas p2 ON c2.nId_Persona = p2.nId_Persona  
        -- Excluir auto-evaluación  
        WHERE Evaluador.nId_Colaborador <> Evaluado.nId_Colaborador  
        AND c1.nEstado_Colaborador = 1  
        AND c2.nEstado_Colaborador = 1;  
    END  
  
  
  
   SELECT *  
   FROM #Evaluators  
   ORDER BY nId_Evaluator, nId_FormType  
END  
go  
  
alter   PROCEDURE [dbo].[sp_smart_evaluation_get_forms_questions]  
(  
    @nId_Evaluation INT,  
    @nId_Form_Type INT  
)  
AS  
BEGIN  
    SELECT       *  
    FROM list_evaluation_questions_options leq  
    WHERE  
       leq.nId_Evaluation = @nId_Evaluation AND  
       leq.nId_Form_Type = @nId_Form_Type  
END  
go  
  
alter   PROCEDURE [dbo].[sp_smart_evaluation_get_forms_resume]  
(  
    @nId_Evaluation INT,  
    @sEvaluator_Name VARCHAR(MAX) = NULL,  
    @sFilterOne VARCHAR(MAX) = NULL --FILTRO POR EQUIPO  
  
)  
AS  
BEGIN  
    DECLARE @script AS nvarchar(max)  
    DECLARE @whereClause AS nvarchar(max) = ''  
    DECLARE @conditions AS TABLE (condition nvarchar(max))  
    DECLARE @conditionOUT AS nvarchar(max);  
  
    IF @sEvaluator_Name IS NOT NULL  
    BEGIN       EXEC sp_get_like_condition 'e.sEvaluator_Name', @sEvaluator_Name, @conditionOUT OUTPUT  
        INSERT INTO @conditions VALUES (@conditionOUT);  
    END  
  
    IF @sFilterOne IS NOT NULL  
    BEGIN       DECLARE @CollaboratorsListTeam VARCHAR(MAX);  
       SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)  
       INSERT INTO @conditions VALUES(N'e.nId_Evaluator in('+@CollaboratorsListTeam+')')  
    END  
  
    --ADD CONDITIONS  
    DECLARE @count INT  
  
    SELECT @count = COUNT(*) FROM @conditions  
  
    SET @whereClause = ' '  
  
    WHILE @count > 0  
    BEGIN  
       DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)  
  
       SET @whereClause = concat (@whereClause,' and ', @condition_temp)  
  
       DELETE TOP (1) FROM @conditions  
       SELECT @count = COUNT(*) FROM @conditions  
    END  
  
    SET @script= N'  
       SELECT           ec.nId_Evaluation,           COUNT(CASE WHEN ec.nAnswered = 0 THEN 1 END) AS nPending,           COUNT(CASE WHEN ec.nAnswered = 1 THEN 1 END) AS nAnswered,          (             SELECT                ft.nId_Form_Type,                ft.sName,                ft.sIcon,                COUNT(CASE WHEN e.nAnswered = 1 THEN 1 END) AS nAnswered,                COUNT(CASE WHEN e.nAnswered = 0 THEN 1 END) AS nPending,                COUNT(CASE WHEN e.nAnswered = 1 THEN 1 END) +                COUNT(CASE WHEN e.nAnswered = 0 THEN 1 END) AS nTotal             FROM v_list_evaluators_form_completition_status e             JOIN Form_Type ft on e.nId_Form_Type = ft.nId_Form_Type             WHERE e.nId_Evaluation = ec.nId_Evaluation ' + @whereClause+ '  
             GROUP BY                ft.nId_Form_Type,                ft.sName,                ft.sIcon             FOR JSON PATH          ) as sResume       FROM (           select               e.nId_Evaluator,               e.nId_Evaluation,               MIN(e.nAnswered) AS nAnswered           from v_list_evaluators_form_completition_status e          WHERE e.nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation) + @whereClause +'  
           group by e.nId_Evaluator,               e.nId_Evaluation       ) ec       GROUP BY ec.nId_Evaluation    '  
    EXECUTE sp_executesql @script  
  
END  
go  
  
alter   PROCEDURE [dbo].[sp_smart_evaluation_get_formtype_details]  
(  
    @nId_Evaluation INT,  
    @nId_Form_Type INT,  
    @sEvaluator_Name VARCHAR(MAX) = NULL,  
    @sFilterOne VARCHAR(MAX) = NULL,--FILTRO POR EQUIPO  
    @sFilterTwo VARCHAR(MAX) = NULL, --FILTRO POR ESTADO  
    @pnum INT = NULL, --Número de Páginas  
    @size INT = NULL --Cantidad de Registros por página  
)  
AS  
BEGIN  
    DECLARE @script AS nvarchar(max)  
    DECLARE @whereClause AS nvarchar(max) = ''  
    DECLARE @paginationClause AS nvarchar(max)  
    DECLARE @conditions AS TABLE (condition nvarchar(max))  
    DECLARE @conditionOUT AS nvarchar(max);  
    DECLARE @conditionState AS nvarchar(max);  
  
  
    IF @sEvaluator_Name IS NOT NULL  
    BEGIN       EXEC sp_get_like_condition 'e.sEvaluator_Name', @sEvaluator_Name, @conditionOUT OUTPUT  
        INSERT INTO @conditions VALUES (@conditionOUT);  
    END  
  
    IF @sFilterOne IS NOT NULL  
    BEGIN       DECLARE @CollaboratorsListTeam VARCHAR(MAX);  
       SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)  
       INSERT INTO @conditions VALUES(N'e.nId_Evaluator in('+@CollaboratorsListTeam+')')  
    END  
  
    IF @sFilterTwo IS NOT NULL  
    BEGIN       SET @conditionState = N'WHERE a.nAnswered in('+@sFilterTwo+')'  
    END  
  
    --ADD CONDITIONS  
    DECLARE @count INT  
  
    SELECT @count = COUNT(*) FROM @conditions  
  
    SET @whereClause = ' '  
  
    WHILE @count > 0  
    BEGIN  
       DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)  
  
       SET @whereClause = concat (@whereClause,' and ', @condition_temp)  
  
       DELETE TOP (1) FROM @conditions  
       SELECT @count = COUNT(*) FROM @conditions  
    END  
  
    --PAGINATION  
    IF @pnum IS NULL AND @size IS NULL  
    BEGIN        SET @paginationClause = ''  
    END  
    ELSE    BEGIN        EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT  
    END  
    SET @script= N'  
       SELECT *       FROM          (             SELECT                e.nId_Evaluator,                e.sEvaluator_Name,                CASE                   WHEN e.nAnswered = 0 THEN 2                   ELSE e.nAnswered                END AS nAnswered,                COUNT(*) OVER() AS nTotal_Registers             FROM v_list_evaluators_form_completition_status e             WHERE e.nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation)+ '  
                and e.nId_Form_Type = '+ CONVERT(NVARCHAR(MAX), @nId_Form_Type)+ @whereClause +'  
          ) a          '+@conditionState+'  
          ORDER BY a.nId_Evaluator ' + @paginationClause  
    EXECUTE sp_executesql @script  
END  
go  
  
CREATE   PROCEDURE sp_smart_evaluation_get_formtype_details_footer(  
    @nId_Evaluation INT,  
    @nId_Form_Type INT,  
    @sEvaluator_Name VARCHAR(MAX) = NULL,  
    @sFilterOne VARCHAR(MAX) = NULL,--FILTRO POR EQUIPO  
    @sFilterTwo VARCHAR(MAX) = NULL --FILTRO POR ESTADO  
)  
AS  
BEGIN  
    DECLARE @script AS nvarchar(max)  
    DECLARE @whereClause AS nvarchar(max) = ''  
    DECLARE @paginationClause AS nvarchar(max)  
    DECLARE @conditions AS TABLE (condition nvarchar(max))  
    DECLARE @conditionOUT AS nvarchar(max);  
  
    IF @sEvaluator_Name IS NOT NULL  
    BEGIN       EXEC sp_get_like_condition 'e.sEvaluator_Name', @sEvaluator_Name, @conditionOUT OUTPUT  
        INSERT INTO @conditions VALUES (@conditionOUT);  
    END  
  
    IF @sFilterOne IS NOT NULL  
    BEGIN       DECLARE @CollaboratorsListTeam VARCHAR(MAX);  
       SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterOne),0,0)  
       INSERT INTO @conditions VALUES(N'e.nId_Evaluator in('+@CollaboratorsListTeam+')')  
    END  
  
    IF @sFilterTwo IS NOT NULL  
    BEGIN       INSERT INTO @conditions VALUES(N'e.nAnswered in('+@sFilterTwo+')')  
    END  
  
    --ADD CONDITIONS  
    DECLARE @count INT  
  
    SELECT @count = COUNT(*) FROM @conditions  
  
    SET @whereClause = ' '  
  
    WHILE @count > 0  
    BEGIN  
       DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)  
  
       SET @whereClause = concat (@whereClause,' and ', @condition_temp)  
  
       DELETE TOP (1) FROM @conditions  
       SELECT @count = COUNT(*) FROM @conditions  
    END  
  
    SET @script= N'  
       SELECT          COUNT(CASE WHEN e.nAnswered = 1 THEN 1 END) AS nAnswered,          COUNT(CASE WHEN e.nAnswered = 0 THEN 1 END) AS nPending       FROM v_list_evaluators_form_completition_status e       WHERE e.nId_Evaluation = '+ CONVERT(NVARCHAR(MAX), @nId_Evaluation)+ '  
          and e.nId_Form_Type = '+ CONVERT(NVARCHAR(MAX), @nId_Form_Type)+ @whereClause  
  
    EXECUTE sp_executesql @script  
  
END  
go  
  
revoke execute on sp_smart_get_Module_Configurations from rol_QA  
go  
  
revoke execute on sp_smart_get_Permit_Closure from rol_QA  
go  
  
revoke execute on sp_smart_get_Permit_Closure_footer from rol_QA  
go  
  
revoke execute on sp_smart_get_asistencia_reporte_diario from rol_QA  
go  
  
revoke execute on sp_smart_get_asistencias from rol_QA  
go  
  
revoke execute on sp_smart_get_asistencias_footer from rol_QA  
go  
  
revoke execute on sp_smart_get_asistencias_vista_cliente from rol_QA  
go  
  
revoke execute on sp_smart_get_attendance_report from rol_QA  
go  
  
revoke execute on sp_smart_get_celebraciones from rol_QA  
go  
  
revoke execute on sp_smart_get_cierre_asistencias from rol_QA  
go  
  
revoke execute on sp_smart_get_client_mg_services from rol_QA  
go  
  
revoke execute on sp_smart_get_client_tasks from rol_QA  
go  
  
revoke execute on sp_smart_get_client_tasks_new from rol_QA  
go  
  
revoke execute on sp_smart_get_collaborators from rol_QA  
go  
  
revoke execute on sp_smart_get_collaborators_footer from rol_QA  
go  
  
revoke execute on sp_smart_get_collaborators_report from rol_QA  
go  
  
revoke execute on sp_smart_get_configurations_list from rol_QA  
go  
  
revoke execute on sp_smart_get_contratos from rol_QA  
go  
  
revoke execute on sp_smart_get_descuentos_solicitudes from rol_QA  
go  
  
revoke execute on sp_smart_get_email_login_super from rol_QA  
go  
  
revoke execute on sp_smart_get_estimated_project_time from rol_QA  
go  
  
revoke execute on sp_smart_get_horarios from rol_QA  
go  
  
revoke execute on sp_smart_get_mailing_templates from rol_QA  
go  
  
revoke execute on sp_smart_get_notificaciones from rol_QA  
go  
  
revoke execute on sp_smart_get_notification_incompleted_tasks from rol_QA  
go  
  
revoke execute on sp_smart_get_plantillas_contratos from rol_QA  
go  
  
revoke execute on sp_smart_get_programmed_emails from rol_QA  
go  
  
revoke execute on sp_smart_get_proy_notification_hours_exceeded from rol_QA  
go  
  
revoke execute on sp_smart_get_proyectos from rol_QA  
go  
  
revoke execute on sp_smart_get_proyectos_footer from rol_QA  
go  
  
revoke execute on sp_smart_get_proyectos_refact from rol_QA  
go  
  
revoke execute on sp_smart_get_questions_and_answers from rol_QA  
go  
  
revoke execute on sp_smart_get_requerimientos from rol_QA  
go  
  
revoke execute on sp_smart_get_requerimientos_refact from rol_QA  
go  
  
revoke execute on sp_smart_get_roles from rol_QA  
go  
  
revoke execute on sp_smart_get_solicitudes from rol_QA  
go  
  
revoke execute on sp_smart_get_solicitudes_Footer from rol_QA  
go  
  
revoke execute on sp_smart_get_solicitudes_cierre_Footer from rol_QA  
go  
  
revoke execute on sp_smart_get_solicitudes_reporte from rol_QA  
go  
  
revoke execute on sp_smart_get_task from rol_QA  
go  
  
revoke execute on sp_smart_get_task_Download from rol_QA  
go  
  
revoke execute on sp_smart_get_task_Download_team_project_view from rol_QA  
go  
  
revoke execute on sp_smart_get_task_Footer from rol_QA  
go  
  
revoke execute on sp_smart_get_task_footer_for_team_project_view from rol_QA  
go  
  
revoke execute on sp_smart_get_task_footer_new from rol_QA  
go  
  
alter     PROCEDURE [dbo].[sp_smart_get_task_new]  
(  
    @nId_User INT,  
    @sTipo_listado nvarchar(max),  
    -----------------------------------------------  
    @SNombreColaborador nvarchar(max),  
    @SNombreProyecto nvarchar(max),  
    @SNombreRequerimiento nvarchar(max),  
    @SNombreCategorias nvarchar(max),  
    @SNombreClientes nvarchar(max),  
    @SNombreServicios nvarchar(max),  
    @SIds nvarchar(max),  
    @fechaIni VARCHAR(MAX),  
    @fechaFin VARCHAR(MAX),  
    -----------------------------------------------  
    @bEnRevision BIT = NULL,  
    -----------------------------------------------  
    @sFilterOne NVARCHAR(MAX), --Filtro por Cliente  
    @sFilterTwo nvarchar(max), --Filtro por tipo Servicio  
    @sFilterThree NVARCHAR(max), --Filtro por Proyecto  
    @sFilterFour NVARCHAR(MAX), --Filtro por tipo de horas  
    @sFilterFive NVARCHAR(MAX), --Filtro por estado de la tarea  
    @sFilterSix NVARCHAR(MAX), --Filtro requerimiento  
    @sFilterSeven NVARCHAR(MAX), --Filtro por categoria  
    @sFilterNotification NVARCHAR(MAX), --Filtro por Id de tarea, notification  
    @sFilterNine NVARCHAR(MAX) = NULL, --Filtro por equipos  
    -----------------------------------------------    @pnum INT = NULL, --N�mero de P�ginas  
    @size INT = NULL, --Cantidad de Registros por p�gina  
    @campo VARCHAR(MAX), --obligatorio column name toOrder  
    @order VARCHAR(MAX)  --obligatorio asc || desc  
)  
AS  
BEGIN  
    DECLARE @script AS nvarchar(max)  
    DECLARE @viewClause AS nvarchar(max)  
    DECLARE @whereClause AS nvarchar(max)  
    DECLARE @orderClause AS nvarchar(max)  
    DECLARE @paginationClause AS nvarchar(max)  
    DECLARE @contador AS nvarchar(max)  
    DECLARE @nId_Colaborador INT = (SELECT c.nId_Colaborador FROM Colaboradores c  
                            JOIN Personas p ON p.nId_Persona = c.nId_Persona  
                            JOIN Usuarios u ON u.nId_Persona = p.nId_Persona  
                            WHERE u.nId_Usuario = @nId_User)  
  
  
    IF @sTipo_listado = 'TASK-LIST-INDIVIDUAL'  
       BEGIN  
          SET @viewClause = N'select * from (select * from v_Listado_Tareas where nId_Usuario ='+CONVERT(varchar,@nId_User)+')a'  
       END  
    ELSE IF @sTipo_listado = 'TASK-LIST-ALL'  
        BEGIN  
            SET @viewClause = N'select * from v_Listado_Tareas'  
        END  
  
    --Set @whereClause  
    DECLARE @conditions AS TABLE (condition nvarchar(max))  
    DECLARE @conditionOUT AS nvarchar(max)  
  
    IF @SIds IS NOT NULL  
       BEGIN           INSERT INTO @conditions VALUES (N'nId IN  (' + @SIds + ')')  
       END  
    ELSE    IF @sFilterNotification IS NOT NULL  
       BEGIN           INSERT INTO @conditions VALUES (N'nId IN  (' + @sFilterNotification + ')')  
       END  
    ELSE    BEGIN        IF @bEnRevision = 0 -- TASKS THAT ARE NOT IN ANOTHER REVISION  
        BEGIN  
         INSERT INTO @conditions VALUES (N'nId_Revision is NULL')  
        END  
  
        IF @fechaIni IS NOT NULL AND @fechaFin IS NOT NULL  
           BEGIN              EXEC sp_get_dates_condition 'dFecha_Registro', @fechaIni ,@fechaFin, @conditionOUT OUTPUT  
             INSERT INTO @conditions VALUES(@conditionOUT)  
           END  
  
       IF @sFilterThree IS NOT NULL  
           BEGIN               INSERT INTO @conditions VALUES (N'sCodigo_Proyecto IN (' + @sFilterThree + ')')  
           END  
  
        IF @sFilterOne IS NOT NULL  
           BEGIN               INSERT INTO @conditions VALUES (N'nId_Cliente IN (' + @sFilterOne + ')')  
           END  
  
        IF @sFilterTwo IS NOT NULL  
           BEGIN               INSERT INTO @conditions VALUES (N'sTipo_Servicio IN (' + @sFilterTwo + ')')  
           END  
  
        IF @sFilterFour IS NOT NULL  
           BEGIN               INSERT INTO @conditions VALUES (N'sCodigo_Tipo_Horas IN (' + @sFilterFour + ')')  
           END  
  
        IF @sFilterFive IS NOT NULL  
           BEGIN               INSERT INTO @conditions VALUES (N'sEstado IN (' + @sFilterFive + ')')  
           END  
  
        IF @sFilterSix IS NOT NULL  
           BEGIN               INSERT INTO @conditions VALUES (N'sCodigo_Requerimiento IN (' + @sFilterSix + ')')  
           END  
  
       IF @sFilterSeven IS NOT NULL  
           BEGIN               INSERT INTO @conditions VALUES (N'sCodigo_Categoria IN (' + @sFilterSeven + ')')  
           END  
  
       --  
       IF @sFilterNine IS NOT NULL  
          BEGIN             DECLARE @CollaboratorsListTeam VARCHAR(MAX);  
             SET @CollaboratorsListTeam = dbo.fn_get_collaborators_by_supervisor(CONVERT(NVARCHAR(MAX), @sFilterNine),0,0)  
             INSERT INTO @conditions VALUES(N'nId_Colaborador in('+@CollaboratorsListTeam+')')  
          END  
       --  
  
        IF @SNombreColaborador IS NOT NULL  
           BEGIN              EXEC sp_get_like_condition 'sNombre_Colaborador', @SNombreColaborador, @conditionOUT OUTPUT  
               INSERT INTO @conditions VALUES (@conditionOUT)  
           END  
  
        IF @SNombreProyecto IS NOT NULL  
           BEGIN              EXEC sp_get_like_condition 'sNombre_Proyecto', @SNombreProyecto, @conditionOUT OUTPUT  
               INSERT INTO @conditions VALUES (@conditionOUT)  
           END  
  
        IF @SNombreRequerimiento IS NOT NULL  
           BEGIN              EXEC sp_get_like_condition 'sNombre_Requerimiento', @SNombreRequerimiento, @conditionOUT OUTPUT  
               INSERT INTO @conditions VALUES (@conditionOUT)  
           END  
  
        IF @SNombreCategorias IS NOT NULL  
           BEGIN              EXEC sp_get_like_condition 'sNombre_Categoria', @SNombreCategorias, @conditionOUT OUTPUT  
               INSERT INTO @conditions VALUES (@conditionOUT)  
           END  
  
        IF @SNombreClientes IS NOT NULL  
           BEGIN              EXEC sp_get_like_condition 'sNombre_Cliente', @SNombreClientes, @conditionOUT OUTPUT  
               INSERT INTO @conditions VALUES (@conditionOUT)  
           END  
  
       IF @SNombreServicios IS NOT NULL  
           BEGIN              EXEC sp_get_like_condition 'sNombreTipoServicio', @SNombreServicios, @conditionOUT OUTPUT  
               INSERT INTO @conditions VALUES (@conditionOUT)  
           END  
       --IF @sFilterNotification IS NOT NULL  
         --  BEGIN           --    INSERT INTO @conditions VALUES (N'nId = ' + @sFilterNotification);           --END    END  
  
    DECLARE @count INT  
  
    SELECT @count = COUNT(*) FROM @conditions  
    if @count > 0  
       begin  
          set @whereClause = N'where '  
       end  
  
    WHILE @count > 0  
       BEGIN  
          DECLARE @condition_temp VARCHAR(max) = (SELECT TOP(1) condition FROM @conditions)  
          IF @count = 1  
             BEGIN  
                SET @whereClause = concat (@whereClause,@condition_temp)  
             END  
          ELSE             BEGIN                SET @whereClause = concat (@whereClause,@condition_temp,' and ')  
             END  
          DELETE TOP (1) FROM @conditions  
          SELECT @count = COUNT(*) FROM @conditions  
       END  
    --END Set @whereClause  
  
    --Set @orderClause    EXEC sp_get_order_clause @campo, @order , 'dFecha_Registro' , @orderClause OUTPUT  
    --END Set @orderClause  
  
    --Set @paginationClause    IF @pnum IS NULL AND @size IS NULL  
    BEGIN        SET @paginationClause = ''  
    END  
    ELSE    BEGIN        EXEC sp_get_pagination_clause @pnum, @size , @paginationClause OUTPUT  
    END    --END Set @paginationClause  
  
    --set @contador-registros    DECLARE @total INT  
    DECLARE @ParmDefinition NVARCHAR(max)  
  
    set @contador = concat('select @totalOut = count(*) from (',@viewClause,' ',@whereClause,')a')  
    SET @ParmDefinition = N'@totalOut int OUTPUT'  
  
    EXEC sp_executesql @contador, @ParmDefinition, @totalOut = @total OUTPUT  
    --end @contador  
  
    --Set @script    SET @script =  concat ('select *,',@total,' as nTotal from (',@viewClause,' ',@whereClause,')a ',@orderClause,' ',@paginationClause)  
    --END Set @script  
  
    EXECUTE sp_executesql @script  
END  
go  
  
revoke execute on sp_smart_get_task_new from rol_QA  
go  
  
revoke execute on sp_smart_get_task_team_project_grouped from rol_QA  
go  
  
revoke execute on sp_smart_get_task_team_project_view from rol_QA  
go  
  
revoke execute on sp_smart_get_treasury_report from rol_QA  
go  
  
revoke execute on sp_smart_get_treasury_request from rol_QA  
go  
  
revoke execute on sp_smart_get_treasury_request_footer from rol_QA  
go  
  
revoke execute on sp_smart_get_users from rol_QA  
go  
  
revoke execute on sp_smart_get_usuarios from rol_QA  
go  
  
revoke execute on sp_smart_get_version_image from rol_QA  
go  
  
revoke execute on sp_smart_mg_services_get_document_of_client from rol_QA  
go  
  
revoke execute on sp_smart_mg_services_get_person_of_client from rol_QA  
go  
  
revoke execute on sp_telephone_get_detail_by_id from rol_QA  
go  
  
revoke execute on sp_telephone_get_detail_by_id_persona from rol_QA  
go  
  
  
    CREATE PROCEDURE dbo.sp_upgraddiagrams  
    AS  
    BEGIN       IF OBJECT_ID(N'dbo.sysdiagrams') IS NOT NULL  
          return 0;  
  
       CREATE TABLE dbo.sysdiagrams  
       (  
          name sysname NOT NULL,  
          principal_id int NOT NULL, -- we may change it to varbinary(85)  
          diagram_id int PRIMARY KEY IDENTITY,  
          version int,  
  
          definition varbinary(max)  
          CONSTRAINT UK_principal_name UNIQUE  
          (  
             principal_id,  
             name  
          )  
       );  
  
  
       /* Add this if we need to have some form of extended properties for diagrams */  
       /*       IF OBJECT_ID(N'dbo.sysdiagram_properties') IS NULL       BEGIN          CREATE TABLE dbo.sysdiagram_properties          (             diagram_id int,             name sysname,             value varbinary(max) NOT NULL          )       END       */  
       IF OBJECT_ID(N'dbo.dtproperties') IS NOT NULL  
       begin          insert into dbo.sysdiagrams  
          (  
             [name],  
             [principal_id],  
             [version],  
             [definition]  
          )  
          select  
             convert(sysname, dgnm.[uvalue]),  
             DATABASE_PRINCIPAL_ID(N'dbo'),       -- will change to the sid of sa  
             0,                   -- zero for old format, dgdef.[version],  
             dgdef.[lvalue]  
          from dbo.[dtproperties] dgnm  
             inner join dbo.[dtproperties] dggd on dggd.[property] = 'DtgSchemaGUID' and dggd.[objectid] = dgnm.[objectid]  
             inner join dbo.[dtproperties] dgdef on dgdef.[property] = 'DtgSchemaDATA' and dgdef.[objectid] = dgnm.[objectid]  
  
          where dgnm.[property] = 'DtgSchemaNAME' and dggd.[uvalue] like N'_EA3E6268-D998-11CE-9454-00AA00A3F36E_'  
          return 2;  
       end  
       return 1;  
    END  
go  
  
exec sp_addextendedproperty 'microsoft_database_tools_support', 1, 'SCHEMA', 'dbo', 'PROCEDURE', 'sp_upgraddiagrams'  
go
```