```sql
  
create or replace view v_collaborator_list AS  
SELECT  
    c.nid_colaborador AS nid,  
    p.spersona_nombre AS snombre_colaborador,  
    d.ntipo_documento,  
    CONCAT(c_doc.sdescripcion, ' ', d.snumero_documento) AS sdocumento_identidad,  
    c_doc.sdescripcion as stipo_documento,  
    d.snumero_documento,  
    c.ddatetime_creador AS dfec_registro,  
    c.dfec_ingreso AS dfec_ingreso,  
    p.dfecha_nacimiento::timestamp AS dfecha_nacimiento,  
    t.stelefono AS ntelefono,  
    c.ntipo_modalidad,  
    s.sdescripcion AS snombre_sede,  
    s.nid_sede AS nid_sede,  
    C6.snombre AS snombre_cargo,  
    C6.nid_cargo AS nid_cargo,  
    p2.spersona_nombre AS snombre_empresa,  
    eu.nid_empresa AS nid_empresa,  
    c.nestado_colaborador AS nestado,  
    c2.sdescripcion AS snombre_estado,  
    c3.sdescripcion AS snombre_modalidad,  
    c.ddatetime_delete AS dfecha_cese,  
    c.surl_foto,  
    (  
        SELECT  
            json_agg(  
                json_build_object(  
                    'sNombre_Supervisor', CONCAT(percol.sprimer_nombre, ' ', percol.sape_paterno),  
                    'nId_Supervisor',supcol.nid_supervisor,  
                    'nOrden',supcol.norden  
                )  
            )  
        FROM fn_get_supervisors_by_collaborators(c.nid_colaborador::text, NULL) supcol  
                 JOIN colaboradores supcolinfo ON supcol.nid_supervisor = supcolinfo.nid_colaborador  
                 JOIN personas percol ON percol.nId_persona = supcolinfo.nid_persona  
    ) AS supervisoresjson  
FROM colaboradores c  
LEFT JOIN personas p ON p.nid_persona = c.nid_persona  
LEFT JOIN documentos d ON d.nid_persona = p.nid_persona AND d.nprincipal = 1  
LEFT JOIN configs c_doc ON d.ntipo_documento = c_doc.scodigo::integer AND c_doc.stabla = 'TIPO_DOCUMENTO'  
LEFT JOIN telefonos t ON t.nid_persona = p.nid_persona AND t.nprincipal = 1  
LEFT JOIN sedes_colaboradores sc ON sc.nid_colaborador = c.nid_colaborador  
LEFT JOIN sedes s ON s.nid_sede = sc.nid_sede  
LEFT JOIN cargos_colaboradores cc ON cc.nid_colaborador = c.nid_colaborador  
LEFT JOIN cargos c6 on CC.nid_cargo = C6.nid_cargo  
LEFT JOIN empresas_usuarias eu ON eu.nid_empresa = c.nid_empresa  
LEFT JOIN personas p2 ON p2.nid_persona = eu.nid_persona  
LEFT JOIN configs c2 ON c2.scodigo = c.nEstado_Colaborador::text AND c2.stabla = 'ESTADO_COLABORADOR'  
LEFT JOIN configs c3 ON c3.scodigo = c.ntipo_modalidad::text AND c3.stabla = 'TIPO_MODALIDAD'  
LEFT JOIN usuarios u ON p.nid_persona = u.nid_persona  
LEFT JOIN roles_usuarios ru ON ru.nid_usuario = u.nid_usuario  
WHERE ru.nid_rol NOT IN (1, 15) OR u.nid_usuario IS NULL;
```