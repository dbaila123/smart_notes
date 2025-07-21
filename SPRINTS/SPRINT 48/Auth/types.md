```sql
CREATE TYPE user_login AS(
    nid_usuario integer,
    nid_colaborador integer,
    snombre_rol text,
    semail text,
    sclave text,
    stoken text,
    spersona_nombre text,
    snombres text,
    sape_paterno text,
    sape_materno text,
    dfecha_nacimiento date,
    nid_rol integer,
    nid_cargo integer,
    nestado_usuario integer,
    nestado_colaborador integer,
    ntipo_autenticacion integer)