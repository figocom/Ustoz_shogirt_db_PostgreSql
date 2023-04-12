# Ustoz_shogirt_db_PostgreSql


create schema if not exists crud;
create schema if not exists helper;
create schema if not exists mapper;
create schema if not exists logs;
set search_path to public;
set time zone 'Asia/Tashkent';
CREATE TYPE customer_status AS ENUM ('BLOCKED', 'ADMIN', 'USER' , 'SUPER_ADMIN');
CREATE TYPE application_status AS ENUM ( 'ACTIVE', 'DELETED', 'NEW','REJECTED', 'ACCEPTED' );
CREATE TYPE helper_status AS ENUM ( 'ACTIVE', 'DELETED', 'UPDATED' );
create type crud.resume_dto as
(
    filename  TEXT,
    file_size int8,
    file_type TEXT,
    file_path TEXT,
    status    helper_status
);
create type crud.application_create_dto as
(
    customer_chat_id varchar,
    title            TEXT,
    context          jsonb
);
create type crud.application_update_dto as
(
    application_id bigint,
    title          TEXT,
    context        jsonb,
    channel_id     bigint,
    post_planned   timestamptz,
    status         application_status
);
create table if not exists application
(
    id_application BIGSERIAL,
    customer_id    BIGINT             not null,
    title          TEXT               not null,
    context        jsonb              not null,
    channel_id     BIGINT             not null,
    post_planned   timestamptz        not null unique,                 -- chiqishidan oldin reja qilingan vaqti
    post_published timestamptz,                                        -- elon chiqqan vaqti
    post_deleted   timestamptz,                                        -- elon o`chirilgan vaqti
    status         application_status not null default 'NEW',          -- A-Active, D-Deleted, H-Hold etc.
    created_at     timestamptz        not null default now(),
    update_at      timestamptz,
    updated_by     bigint,
    deleted_at     timestamptz,
    deleted_by     bigint,
    is_deleted     boolean                     default false not null, -- Default false, True=if deleted
    primary key (id_application)

);
create table if not exists customer
(
    id_customer BIGSERIAL,
    chat_id     varchar unique not null,                -- Telegram user id
    status      customer_status         default 'USER', -- A-Active, D-Deleted, H-Hold etc.
    created_at  timestamptz    not null default now(),
    updated_by  bigint,
    update_at   timestamptz,
    deleted_at  timestamptz,
    deleted_by  bigint,
    is_deleted  boolean        not null default false,  -- Default false, True=if deleted
    primary key (id_customer)
);
create table if not exists resume
(
    id_resume   BIGSERIAL,
    customer_id BIGINT        not null,
    filename    TEXT          not null,-- real filename.
    file_size   int8          not null,-- File size in byte
    file_type   TEXT          not null, -- pdf, jpg and other
    file_path   TEXT          not null,-- filename in folder
    status      helper_status not null default 'ACTIVE',-- A-Active, D-Deleted, H-Hold etc.
    created_at  timestamptz   not null default now(),
    update_at   timestamptz,
    updated_by  bigint,
    deleted_at  timestamptz,
    is_deleted  boolean                default false not null, -- Default false, True=if deleted
    primary key (id_resume),
    unique (customer_id,filename)

);
create table if not exists category
(
    id_category BIGSERIAL,
    "name"      TEXT          not null unique,
    is_customer boolean                default false,
    customer_id BIGINT        not null,                        -- Agar is_custome=true bo`lsa, kim qo`shgani ko`rsatiladi. Aks holda NULL
    status      helper_status not null default 'ACTIVE',       -- A-Active, D-Deleted, H-Hold etc.
    created_at  timestamptz   not null default now(),
    updated_by  bigint,
    update_at   timestamptz,
    deleted_at  timestamptz,
    deleted_by  bigint,
    is_deleted  boolean                default false not null, -- Default false, True=if deleted
    primary key (id_category)
);


create table if not exists technology
(
    id_technology BIGSERIAL,
    "name"        TEXT          not null unique,
    category_id   bigint        not null,
    is_customer   boolean                default false,    -- foydalanuvchi yangi qo`shgan texnologiya bo`lsa = True
    customer_id   BIGINT        not null,                  -- Agar is_custome=true bo`lsa, kim qo`shgani ko`rsatiladi. Aks holda NULL
    status        helper_status not null default 'ACTIVE', -- A-Active, D-Deleted, H-Hold etc.
    created_at    timestamptz   not null default now(),
    update_at     timestamptz,
    updated_by    bigint,
    deleted_at    timestamptz,
    deleted_by    bigint,
    is_deleted    boolean       not null default false,    -- Default false, True=if deleted
    primary key (id_technology)
);

create table if not exists application_technology
(
    id_application_technology BIGSERIAL,
    application_id            BIGINT,
    technology_id             BIGINT,
    status                    helper_status not null default 'ACTIVE',      -- A-Active, D-Deleted, H-Hold etc.
    created_at                timestamptz   not null default now(),
    update_at                 timestamptz,
    updated_by                bigint,
    deleted_at                timestamptz,
    is_deleted                boolean                default false not null -- Default false, True=if deleted
);
create table if not exists telegram_cmd
(
    id_telegram_cmd   BIGSERIAL,
    message_type      TEXT        not null,-- message, callback_query,
    update_id         BIGINT      not null,
    messageId         BIGINT      not null,
    message_date      timestamptz not null default now(),
    "message_text"    TEXT,-- foydalanuvchi yuborgan matn
    message_body      jsonb,-- bu webhook orqali kelgan to`liq bodyni o`zi
    first_name        TEXT,-- foydalanuvchi ismi. panelda keyin ko`rsatamiz.
    chat_id_from      BIGINT,
    chat_id_to        BIGINT,
    is_bot            boolean,-- agar bot yuborgan bo`lsa, TRUE bo`lishi kerak
    is_admin          boolean,-- agar configda ko`rsatilgan adminlar yuborgan bo'lsa TRUE
    username          TEXT,-- agar username qo`ygan bo`lsa, o`sha nom
    callback_query_id BIGINT,
    callback_data     TEXT,
    status            helper_status        default 'ACTIVE',-- A-Active, D-Deleted, H-Hold etc.
    created_at        timestamptz          default now(),
    update_at         timestamptz,
    deleted_at        timestamptz,
    is_deleted        boolean              default false,-- Default false, True=if deleted
    primary key (id_telegram_cmd)
);
create table if not exists telegram_photo
(
    id_telegram_photo BIGSERIAL,
    telegram_cmd_id   BIGINT,
    file_unique_id    TEXT,
    file_size         BIGINT,
    width             integer,
    height            integer,
    photo_url         TEXT,                                    -- file_id - telegram serveridagi URL
    photo_path        TEXT,                                    -- saqlab olingan joni
    status            helper_status not null default 'ACTIVE', -- A-Active, D-Deleted, H-Hold etc.
    created_at        timestamptz            default now(),
    update_at         timestamptz,
    deleted_at        timestamptz,
    is_deleted        boolean                default false,-- Default false, True=if deleted
    primary key (id_telegram_photo)
);
ALTER TABLE "application"
    ADD FOREIGN KEY ("customer_id") REFERENCES "customer" ("id_customer");
ALTER TABLE "resume"
    ADD FOREIGN KEY ("customer_id") REFERENCES "customer" ("id_customer");
ALTER TABLE "application_technology"
    ADD FOREIGN KEY ("application_id") REFERENCES "application" ("id_application");
ALTER TABLE "application_technology"
    ADD FOREIGN KEY ("technology_id") REFERENCES "technology" ("id_technology");
ALTER TABLE technology
    ADD FOREIGN KEY (id_technology) REFERENCES category (id_category);
ALTER TABLE "telegram_photo"
    ADD FOREIGN KEY ("telegram_cmd_id") REFERENCES "telegram_cmd" ("id_telegram_cmd");
		
		
create or replace procedure helper.check_dataparam(dataparam text)
    language plpgsql
as
$$
begin
    if dataparam is null or dataparam = '{}'::text then
        raise exception 'Data param is invalid!';
    end if;
end;
$$;


create or replace procedure helper.is_active(i_user_chat_id varchar)
    language plpgsql
as
$$
begin
    if i_user_chat_id is null then
        raise exception 'User not found with this % id',i_user_chat_id;
    end if;
    if not exists(select * from public.customer u where u.chat_id = i_user_chat_id and is_deleted = false) then
        raise exception 'User not found with % this id',i_user_chat_id;
    end if;
    if 'BLOCKED'::public.customer_status =
       (select status from public.customer c where c.chat_id = i_user_chat_id and is_deleted = false) then
        raise exception 'User is blocked';
    end if;
end
$$;


create or replace function helper.isAdmin(i_chat_id varchar) returns boolean
    language plpgsql
as
$$
declare
    t_user record;
BEGIN
    if i_chat_id is null then
        return false;
    end if;
    select * into t_user from public.customer t where t.is_deleted = false and t.chat_id = i_chat_id;
    return FOUND and t_user.status::text ilike 'ADMIN' or t_user.status::text ilike 'SUPER_ADMIN';
end
$$;


create or replace procedure helper.check_for_null_or_length_zero(content text, message character varying)
    language plpgsql
as
$$
BEGIN

    if content is null or
       length(trim(content)) = 0 then
        raise exception '%',message;
    end if;
END;
$$;


create or replace function helper.getCustomerId(i_chat_id varchar) returns bigint
    language plpgsql as
$$
BEGIN
    return (select id_customer from public.customer where chat_id = i_chat_id);
END
$$;


create or replace procedure crud.category_create(category_name text, i_chat_id varchar)
    language plpgsql
as
$$
DECLARE
    isCustomer  boolean;
    id_customer bigint;
BEGIN
    call helper.is_active(i_chat_id);
    call helper.check_for_null_or_length_zero(
            category_name::text, 'Category name is invalid');

    select helper.isAdmin(i_chat_id) into isCustomer;
    if exists(select *
              from public.category c
              where c.name ilike category_name) then
        raise exception 'Category with name ''%'' already exists ', category_name
            using detail = 'Category name should be unique';
    end if;
    select into id_customer helper.getCustomerId(i_chat_id);
    insert into public.category(name, is_customer, customer_id)
    VALUES (category_name, !isCustomer, id_customer);
END
$$;


create or replace procedure crud.user_create(i_chat_id varchar DEFAULT NULL)
    language plpgsql
as
$$
BEGIN
    call helper.check_for_null_or_length_zero(
            i_chat_id::text, 'Chat id is invalid');
    if exists(select *
              from public.customer c
              where c.chat_id ilike i_chat_id) then
        raise exception 'User with chat id ''%'' already exists ', i_chat_id
            using detail = 'User chat id should be unique';
    end if;
    insert into public.customer(chat_id)
    VALUES (i_chat_id);
END
$$;


create or replace procedure crud.technology_create(technology_name text, i_category_id bigint,
                                                   i_chat_id varchar)
    language plpgsql
as
$$
DECLARE
    isCustomer  boolean;
    id_customer bigint;
BEGIN
    call helper.is_active(i_chat_id);
    call helper.check_for_null_or_length_zero(
            technology_name::text, 'Technology name is invalid');
    call helper.check_for_null_or_length_zero(
            i_category_id::text, 'Category id is invalid');
    select helper.isAdmin(i_chat_id) into isCustomer;
    if exists(select *
              from public.technology c
              where c.name ilike technology_name) then
        raise exception 'Technology with name ''%'' already exists ', technology_name
            using detail = 'Technology name should be unique';
    end if;
    if not FOUND then
        raise exception 'Category not found';
    end if;
    select into id_customer helper.getCustomerId(i_chat_id);
    insert into public.technology(name, is_customer, customer_id, category_id)
    VALUES (technology_name, isCustomer, id_customer, i_category_id);
END
$$;



create or replace function mapper.json_to_resume_dto(json_data json) returns crud.resume_dto
    language plpgsql
as
$$
DECLARE
    data crud.resume_dto;
BEGIN
    data.filename := json_data ->> 'filename';
    data.file_size := json_data ->> 'file_size';
    data.file_type := json_data ->> 'file_type';
    data.file_path := json_data ->> 'file_path';
    data.status := json_data ->> 'status';
    return data;
END
$$;


create or replace procedure crud.create_resume(data_params text, i_chat_id varchar)
    language plpgsql
as
$$
DECLARE
    id_customer bigint;
    json_data   json;
    c_dto       crud.resume_dto;
BEGIN
    call helper.check_dataparam(data_params);
    json_data := data_params::json;
    c_dto := mapper.json_to_resume_dto(json_data);
    call helper.is_active(i_chat_id);
    select into id_customer helper.getCustomerId(i_chat_id);
    call helper.check_for_null_or_length_zero(
            c_dto.filename::text, 'File name is invalid');
    call helper.check_for_null_or_length_zero(
            c_dto.file_path::text, 'File path is invalid');
    call helper.check_for_null_or_length_zero(
            c_dto.status::text, 'File status is invalid');
    if exists(select t.file_path from public.resume t where t.file_path = c_dto.file_path) then
        raise exception 'Resume already exist';
    end if;
    insert into public.resume(customer_id, filename, file_size, file_type, file_path, status)
    VALUES (id_customer,
            c_dto.filename,
            c_dto.file_size,
            c_dto.file_type,
            c_dto.status);
END
$$;


create or replace function mapper.json_to_application_create_dto(json_data json) returns crud.application_create_dto
    language plpgsql
as
$$
DECLARE
    data crud.application_create_dto;
BEGIN
    data.customer_chat_id := json_data ->> 'customer_chat_id';
    data.title := json_data ->> 'title';
    data.context := json_data ->> 'context';
    return data;
END
$$;
create or replace function crud.application_create(data_params text) returns bigint
    language plpgsql
as
$$
DECLARE
    id_customer    bigint;
    application_id bigint;
    json_data      json;
    c_dto          crud.application_create_dto;
BEGIN
    call helper.check_dataparam(data_params);
    json_data := data_params::json;
    c_dto := mapper.json_to_application_create_dto(json_data);
    call helper.is_active(c_dto.customer_chat_id);
    select into id_customer helper.getCustomerId(c_dto.customer_chat_id);
    call helper.check_for_null_or_length_zero(
            c_dto.title::text, 'Title is invalid');
    call helper.check_for_null_or_length_zero(
            c_dto.context::text, 'Context path is invalid');
    insert into public.application(customer_id, title, context)
    VALUES (id_customer,
            c_dto.title,
            c_dto.context)
    returning id_application into application_id;
    return application_id;
END
$$;
create or replace procedure crud.application_technology_create(i_technology_id bigint,
                                                               i_application_id bigint)
    language plpgsql
as
$$
BEGIN
    call helper.check_for_null_or_length_zero(
            i_technology_id::text, 'Technology id is invalid');
    call helper.check_for_null_or_length_zero(
            i_application_id::text, 'application id is invalid');
    insert into public.application_technology(application_id, technology_id)
    VALUES (i_application_id, i_technology_id);
END
$$;
create or replace function mapper.json_to_telegram_cmd(json_data json) returns public.telegram_cmd
    language plpgsql
as
$$
DECLARE
    data public.telegram_cmd;
BEGIN
    data.message_type := json_data ->> 'message_type';
    data.update_id := json_data ->> 'update_id';
    data.messageid := json_data ->> 'messageid';
    data.message_text := json_data ->> 'message_text';
    data.message_body := json_data ->> 'message_body';
    data.first_name := json_data ->> 'first_name';
    data.chat_id_from := json_data ->> 'chat_id_from';
    data.chat_id_to := json_data ->> 'chat_id_to';
    data.is_bot := json_data ->> 'is_bot';
    data.is_admin := json_data ->> 'is_admin';
    data.username := json_data ->> 'username';
    data.callback_query_id := json_data ->> 'callback_query_id';
    data.callback_data := json_data ->> 'callback_data';
    return data;
END
$$;
create or replace function crud.telegram_cmd_create(data_params text) returns bigint
    language plpgsql
as
$$
DECLARE
    json_data json;
    dto       public.telegram_cmd;
BEGIN
    call helper.check_dataparam(data_params);
    json_data := data_params::json;
    dto := mapper.json_to_telegram_cmd(json_data);
    insert into public.telegram_cmd(message_type, update_id, messageid, "message_text", message_body, first_name,
                                    chat_id_from,
                                    chat_id_to, is_bot, is_admin, username, callback_query_id, callback_data)
    VALUES (dto.message_type, dto.update_id, dto.messageid, dto.message_text, dto.message_body, dto.first_name,
            dto.chat_id_from, dto.chat_id_to, dto.is_bot, dto.is_admin, dto.username, dto.callback_query_id,
            dto.callback_data)
    returning id_telegram_cmd;
END
$$;
create or replace function mapper.json_to_telegram_photo(json_data json) returns public.telegram_photo
    language plpgsql
as
$$
DECLARE
    data public.telegram_photo;
BEGIN
    data.telegram_cmd_id := json_data ->> 'telegram_cmd_id';
    data.file_unique_id := json_data ->> 'file_unique_id';
    data.file_size := json_data ->> 'file_size';
    data.width := json_data ->> 'width';
    data.height := json_data ->> 'height';
    data.photo_url := json_data ->> 'photo_url';
    data.photo_path := json_data ->> 'photo_path';
    return data;
END
$$;
create or replace procedure crud.telegram_photo_create(data_params text)
    language plpgsql
as
$$
DECLARE
    json_data json;
    dto       public.telegram_photo;
BEGIN
    call helper.check_dataparam(data_params);
    json_data := data_params::json;
    dto := mapper.json_to_telegram_photo(json_data);
    insert into public.telegram_photo(telegram_cmd_id, file_unique_id, file_size, width, height, photo_url, photo_path)
    VALUES (dto.telegram_cmd_id, dto.file_unique_id, dto.file_size, dto.width, dto.height, dto.photo_url,
            dto.photo_path);
END
$$;
create or replace function mapper.json_to_application_update_dto(json_data json) returns crud.application_update_dto
    language plpgsql
as
$$
DECLARE
    data crud.application_update_dto;
BEGIN
    data.application_id := json_data ->> 'application_id';
    data.title := json_data ->> 'title';
    data.context := json_data ->> 'context';
    data.channel_id := json_data ->> 'channel_id';
    data.post_planned := json_data ->> 'post_planned';
    data.status := json_data ->> 'status';
    return data;
END
$$;
create or replace procedure crud.application_update(data_params text, session_user_chat_id varchar)
    language plpgsql
as
$$
DECLARE
    id_customer   bigint;
    t_application record;
    json_data     json;
    dto           crud.application_update_dto;
begin
    call helper.check_dataparam(data_params);
    call helper.is_active(session_user_chat_id);

    if (select helper.isAdmin(session_user_chat_id)) is false then
        raise exception 'Permission denied' using detail = 'Admin can change application';
    end if;
    select into id_customer helper.getCustomerId(session_user_chat_id);
    json_data := data_params::json;
    dto := mapper.json_to_application_update_dto(json_data);

    select *
    into t_application
    from public.application t
    where t.is_deleted = false
      and t.id_application = dto.application_id;
    if dto.post_planned is null then
        dto.post_planned := t_application.post_planned;
    end if;
    if dto.channel_id is null then
        dto.channel_id := t_application.channel_id;
    end if;
    if dto.title is null then
        dto.title := t_application.title;
    end if;
    if dto.context is null then
        dto.context := t_application.context;
    end if;
    update public.application
    set channel_id     = dto.channel_id,
        title          = dto.title,
        post_planned   = dto.post_planned,
        post_published =dto.post_planned,
        context        = dto.context,
        status         = dto.status,
        update_at      =now(),
        updated_by     = id_customer
    where id_application = dto.application_id;
end
$$;
create or replace procedure crud.application_delete(application_id bigint, session_user_chat_id varchar)
    language plpgsql
as
$$
declare
    id_customer bigint;
BEGIN
    call helper.is_active(session_user_chat_id);
    if (select helper.isAdmin(session_user_chat_id)) is false then
        raise exception 'Permission denied';
    end if;
    select into id_customer helper.getCustomerId(session_user_chat_id);
    update public.application
    set is_deleted = true,
        deleted_at =now(),
        deleted_by=id_customer
    where id_application = application_id;
END
$$;
create or replace function helper.get_application_id(i_context text) returns bigint
    language plpgsql
as
$$
begin
    return (select id_application
            from public.application t
            where t.is_deleted = false
              and context::text = i_context);
end
$$;

create or replace function crud.get_application(i_application_id bigint DEFAULT NULL::bigint) returns text
    language plpgsql
as
$$
begin
    return ((select (json_build_object(
            'id_application', t.id_application,
            'customer_id', t.customer_id,
            'title', t.title,
            'context', t.context,
            'channel_id', t.channel_id,
            'post_planned', t.post_planned,
            'post_published', t.post_published,
            'post_deleted"', t."post_deleted",
            'status', t.status,
            'created_at', t.created_at,
            'update_at', t.update_at,
            'updated_by', t.updated_by,
            'deleted_at', t.deleted_at,
            'deleted_by', t.deleted_by,
            'is_deleted', t.is_deleted
        ))
             from public.application t
             where t.id_application = i_application_id)::text);
end
$$;
create or replace function crud.get_all_applications() returns text
    language plpgsql
as
$$
begin
    return coalesce((select json_agg(crud.get_application(t.id_application)::jsonb)
                     from public.application t)::text, '[]');
end
$$;
create or replace procedure crud.category_update(category_name text, category_id bigint, session_user_chat_id varchar)
    language plpgsql
as
$$
DECLARE
    id_customer bigint;
begin
    call helper.is_active(session_user_chat_id);
    if (select helper.isAdmin(session_user_chat_id)) is false then
        raise exception 'Permission denied' using detail = 'Admin can change application';
    end if;
    call helper.check_for_null_or_length_zero(
            category_name, 'category name is invalid');
    if exists(select *
              from public.category c
              where c.name ilike category_name) then
        raise exception 'Category with name ''%'' already exists ', category_name;
    end if;
    select into id_customer helper.getCustomerId(session_user_chat_id);
    update public.category
    set name       = category_name,
        update_at  =now(),
        updated_by = id_customer,
        status     = 'UPDATED'::helper_status
    where id_category = category_id;
end
$$;
create or replace procedure crud.category_delete(category_id bigint, session_user_chat_id varchar)
    language plpgsql
as
$$
declare
    id_customer bigint;
BEGIN
    call helper.is_active(session_user_chat_id);
    if (select helper.isAdmin(session_user_chat_id)) is false then
        raise exception 'Permission denied';
    end if;
    select into id_customer helper.getCustomerId(session_user_chat_id);
    update public.category
    set is_deleted = true,
        deleted_at =now(),
        deleted_by=id_customer
    where id_category = category_id;
END
$$;
create or replace function crud.get_category(i_category_id bigint DEFAULT NULL::bigint) returns text
    language plpgsql
as
$$
begin
    return (select t.name from public.category t where t.id_category = i_category_id and is_deleted = false);
end
$$;
create or replace function crud.get_all_category() returns text
    language plpgsql
as
$$
begin
    return (select array_agg(crud.get_category(t.id_category))
            from public.category t)::text;
end
$$;
create or replace procedure crud.technology_update(technology_name text, technology_id bigint,
                                                   session_user_chat_id varchar)
    language plpgsql
as
$$
DECLARE
    id_customer bigint;
begin
    call helper.is_active(session_user_chat_id);
    if (select helper.isAdmin(session_user_chat_id)) is false then
        raise exception 'Permission denied' using detail = 'Admin can change application';
    end if;
    call helper.check_for_null_or_length_zero(
            technology_name, 'technology name is invalid');
    if exists(select *
              from public.technology c
              where c.name ilike technology_name) then
        raise exception 'technology with name ''%'' already exists ', technology_name;
    end if;
    select into id_customer helper.getCustomerId(session_user_chat_id);
    update public.technology
    set name       = technology_name,
        update_at  =now(),
        updated_by = id_customer,
        status     = 'UPDATED'::helper_status
    where id_technology = technology_id;
end
$$;
create or replace procedure crud.technology_delete(technology_id bigint, session_user_chat_id varchar)
    language plpgsql
as
$$
declare
    id_customer bigint;
BEGIN
    call helper.is_active(session_user_chat_id);
    if (select helper.isAdmin(session_user_chat_id)) is false then
        raise exception 'Permission denied';
    end if;
    select into id_customer helper.getCustomerId(session_user_chat_id);
    update public.technology
    set is_deleted = true,
        deleted_at =now(),
        deleted_by=id_customer
    where id_technology = technology_id;
END
$$;
create or replace function crud.get_technology(i_technology_id bigint DEFAULT NULL::bigint) returns text
    language plpgsql
as
$$
begin
    return (select t.name from public.technology t where t.id_technology = i_technology_id and is_deleted = false);
end
$$;
create or replace function crud.get_all_technology() returns text
    language plpgsql
as
$$
begin
    return (select array_agg(crud.get_technology(t.id_technology))
            from public.technology t)::text;
end
$$;
create or replace procedure crud.user_update(new_status text, customer_id bigint,
                                             session_user_chat_id varchar)
    language plpgsql
as
$$
DECLARE
    id_admin bigint;
begin
    call helper.is_active(session_user_chat_id);
    if (select helper.isAdmin(session_user_chat_id)) is false then
        raise exception 'Permission denied' using detail = 'Admin can change application';
    end if;
    call helper.check_for_null_or_length_zero(
            new_status, 'technology name is invalid');
    select into id_admin helper.getCustomerId(session_user_chat_id);
    update public.customer
    set status     = new_status::customer_status,
        update_at  =now(),
        updated_by = id_admin
    where id_customer = customer_id;
end
$$;
create or replace procedure crud.user_delete(customer_id bigint, session_user_chat_id varchar)
    language plpgsql
as
$$
declare
    id_admin bigint;
BEGIN
    call helper.is_active(session_user_chat_id);
    if (select helper.isAdmin(session_user_chat_id)) is false then
        raise exception 'Permission denied';
    end if;
    select into id_admin helper.getCustomerId(session_user_chat_id);
    update public.customer
    set is_deleted = true,
        deleted_at =now(),
        deleted_by=id_admin
    where id_customer = customer_id;
END
$$;
create or replace function crud.get_customer(i_customer_id bigint DEFAULT NULL::bigint) returns text
    language plpgsql
as
$$
begin
    return (select t.chat_id from public.customer t where t.id_customer = i_customer_id and is_deleted = false);
end
$$;
create or replace function crud.get_all_customer() returns text
    language plpgsql
as
$$
begin
    return (select array_agg(crud.get_customer(t.id_customer))
            from public.customer t)::text;
end
$$;
create or replace procedure crud.resume_delete(resume_id bigint, session_user_chat_id varchar)
    language plpgsql
as
$$
declare
    id_customer bigint;
BEGIN
    call helper.is_active(session_user_chat_id);
    select into id_customer helper.getCustomerId(session_user_chat_id);
    if id_customer = (select customer_id from public.resume where id_resume = resume_id) then
        update public.resume
        set is_deleted = true,
            deleted_at =now()
        where id_resume = resume_id;
    else
        raise exception 'Permission denied';
    end if;
END
$$;
create or replace function crud.get_resume(i_filename text) returns text
    language plpgsql
as
$$
begin
    return (select t.filename from public.resume t where t.filename=i_filename and is_deleted = false);
end
$$;
create or replace function crud.get_all_resume(i_user_id bigint DEFAULT NULL::bigint) returns text
    language plpgsql
as
$$
begin
    return (select array_agg((select t.filename
                              from public.resume t
                              where is_deleted = false and customer_id = i_user_id)))::text;
end
$$;






