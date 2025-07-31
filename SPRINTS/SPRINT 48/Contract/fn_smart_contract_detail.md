```SQL
CREATE OR REPLACE FUNCTION tenant0001_dbo.fn_smart_contract_detail(nid_contract integer)
RETURNS TABLE(
	nid_contrato integer, 
	nid_colaborador integer, 
	spersona_nombre text, 
	nid_cargo integer, 
	snombre_cargo text, 
	nid_empresa integer, 
	snombre_empresa text, 
	dsalario numeric, 
	ntipo_contratacion integer, 
	stipocontratacion text, 
	sfecha_ini date, 
	sfecha_fin date, 
	sobservacion text, 
	scodigo_tipo_documento integer, 
	snumero_documento text, 
	sdireccion text, 
	sdistrito text, 
	sdepartamento text, 
	sprovincia text, 
	shorariodetalle text, 
	surl_file text, 
	nestado_contrato integer
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT
        c.nid_contrato,
        c.nid_colaborador,
        p.spersona_nombre AS snombre_colaborador,
        c.nid_cargo,
        car.sdescripcion AS snombre_cargo,
        co.nid_empresa,
        pe.spersona_nombre AS snombre_empresa,
        c.dsalario,
        co.ntipo_contratacion,
        config.sdescripcion AS stipocontratacion,
        c.dfecha_ini AS sfecha_ini,
        c.dfecha_fin AS sfecha_fin,
        c.sobservacion,
        doc.ntipo_documento AS scodigo_tipo_documento,
        doc.snumero_documento,
        dir.scalle AS sdireccion,
        dep.subigeo_distrito AS sdistrito,
        dep.subigeo_departamento AS sdepartamento,
        dep.subigeo_provincia AS sprovincia,
        h.scomentario AS shorariodetalle,
        (
            SELECT f.surl_file
            FROM files f
            WHERE f.nid_entidad = c.nid_contrato
            LIMIT 1
        ) AS surl_file,
        c.nestado_contrato
    FROM contratos c
    INNER JOIN cargos car ON c.nid_cargo = car.nid_cargo
    INNER JOIN colaboradores co ON c.nid_colaborador = co.nid_colaborador
    INNER JOIN personas p ON co.nid_persona = p.nid_persona
    INNER JOIN documentos doc ON p.nid_persona = doc.nid_persona
    INNER JOIN direcciones dir ON p.nid_persona = dir.nid_persona
    INNER JOIN ubigeos dep ON dir.nid_ubigeo = dep.nid_ubigeo
    INNER JOIN horarios_colaboradores hc ON co.nid_colaborador = hc.nid_colaborador
    INNER JOIN horarios h ON hc.nid_horario = h.nid_horario
    INNER JOIN empresas_usuarias eu ON co.nid_empresa = eu.nid_empresa
    INNER JOIN personas pe ON eu.nid_persona = pe.nid_persona
    INNER JOIN configs config ON co.ntipo_contratacion = config.scodigo
    WHERE c.nid_contrato = nid_contract
      AND config.stabla = 'TIPO_CONTRATACION';
END;
$$;
```