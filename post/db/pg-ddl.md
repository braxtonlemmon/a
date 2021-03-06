---
title: Posstgre shell+ddl
date: 2018-09-27
private:
---
# shell

## connect pg
首次安装后，默认数据库是postgres

    psql postgres

### shell 参数
默认同时支持 unix domain socket + ip/port net socket

    psql DBNAME USERNAME
    psql -U user_name -d database_name -h 127.0.0.1 -W
    psql -U user_name database_name -h 127.0.0.1 -W
        \W prompt enter password(可省略)
    psql postgresql://t1:1@47.96.1.162:6379/template1

### Non interactive password:

1. vim ~/.pgpass:
    `hostname:port:database:username:password`
2. PGPASSWORD=pass1234 psql -U MyUsername myDatabaseName
3. URI: https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-CONNSTRING

e.g.

    psql postgresql://
    psql postgresql://user:secret@localhost:5432/mydb
    psql postgresql://other@localhost/otherdb?connect_timeout=10&application_name=myapp
    psql postgresql://host1:123,host2:456/somedb?target_session_attrs=any&application_name=myapp

.pgpass config(recommend):

    1) Create .pgpass file with content
        host:5432:somedb:someuser:somepass

    2) set the permissions using command
        sudo chmod 600 .pgpass

    3) Set the file owner as the same user using which you logged in(实际不需要) :
        sudo chown login_username:login_username .pgpass

    4) Set PGPASSFILE environment variable :
        export PGPASSFILE='/home/user/.pgpass'

    Now check by connecting to database :
        psql -h host -U someuser somedb

## exec sql

    $ psql -h 127.0.0.1 -p 5930 -c "select 1"
    $ psql -h 127.0.0.1 -p 5930 -f a.sql

### exec sql file
    psql -f exec.sql
    pg_dump dbname > outfile

    psql [dbname] < exec.sql
    cat exec.sql | psql [dbname] 


# import export
## pg_dump
    pg_dump help
### backup

    # only schema
    pg_dump -U db_username -s  -f [filename.sql] [db_name]
    # only data
    pg_dump -U db_username -a  -f [filename.sql] [db_name]
    # spcify table
    pg_dump -U db_username -t table_name -a  -f [filename.sql] [db_name]
    # data+schema
    pg_dump                      -f [filename.sql] [db_name]

    -F format
        -Fc custom, Output a custom-format archive suitable for input
        -Fp plain 默认
    -U username

恢复：

    cat  filename.sql | psql  -U db_username

### restore
https://www.postgresql.org/docs/9.2/app-pgrestore.html

只用于自定的数据的恢复

    $ pg_dump -Fc mydb > db.dump
    $ dropdb mydb
    $ pg_restore -C -d postgres db.dump

custom-format archive:

    # schema
    pg_restore -s -d [db_name] [filename.sql]
    # data
    pg_restore -a -d [db_name] [filename.sql]
    # schema and data
    pg_restore -d [db_name] [filename.sql]

## copy db/table/result
### export db

    $ pg_dump -U username dbname > dbexport.pgsql
    $ psql -U username dbname < dbexport.pgsql

### export table:

    \copy my_table to 'my_table.csv' csv;
    \copy my_table FROM 'my_table.csv' DELIMITER ',' CSV;

### export table with bash:

    $ pg_dump \
    -h localhost \
    -p 5432 \
    -U user -W \
    --table="table-name" --data-only --column-inserts database-name > table.sql

    $ psql \
    -h localhost \
    -p 5432 \
    -U user \
    database-name \
    -f table.sql

### export sql 
relative path 

    \copy (select * from my_table limit 10) TO './a.csv'; -- 空格分析
    \copy (select * from my_table limit 10) TO './a.csv' CSV 
    \copy (select * from my_table limit 10) TO './a.csv' CSV HEADER
    \copy (select * from my_table limit 10) TO './a.csv' HEADER

full path

    \COPY products_273 TO '/tmp/products_199.csv' WITH (FORMAT CSV, HEADER);

    COPY persons TO 'C:\tmp\persons_db.csv' DELIMITER ',' CSV HEADER;

    COPY persons(first_name,last_name,email) TO 'C:\tmp\persons_partial_db.csv' DELIMITER ',' CSV HEADER;

    \copy (Select * From foo) To '/tmp/test.csv' With CSV
    \copy (select * from my_table limit 10) TO '~/Downloads/export.csv' CSV HEADER


# help
`\h CREATE ROLE`
`\? \l`

# crud
## database
help:

    \h alter DATABASE

create:

    $ createdb test1
    > CREATE DATABASE ahuigo;
    > CREATE DATABASE yuzhi100 OWNER myuser;

drop db:

    > drop database yunzhi100

rename db:

    ALTER DATABASE name RENAME TO new_name

list db:

    \l
        list all databases

### connect db:

    \c database_name
        \connect database_name
    SELECT current_database();

### current database

    SELECT current_database();

## table

### show table
describe table and sequence:

    \d
    \dt # with table_squence
    \dt [<table>]

show create table:

    pg_dump -st tablename dbname

### create

    $ psql -f init.sql
    CREATE TABLE playground (
        id serial PRIMARY KEY,
        uid int UNIQUE,
        name varchar (50) NOT NULL,
        location varchar(25) check (location in ('north', 'south', 'west', 'east',  'northwest')),
        install_date date
    );

#### create view table
	create view t_view as
		select s1,s2,t1.id from t1,t2 where t1.id=t2.id order by s2;

	create view t_view_alias (seg1, seg2, id) as
		select s1,s2,t1.id from t1,t2 where t1.id=t2.id order by s2;

### drop

    drop TABLE [IF EXISTS ] xxx
    drop TABLE xxx1,xx2

## Alter
ALTER TABLE table_name `<action>`:

    column:
        ADD COLUMN column_name VARCHAR [not null  default 3];
        DROP COLUMN column_name;
        ALTER COLUMN location TYPE VARCHAR,
            ALTER COLUMN asset_no TYPE INT USING asset_no::integer;
            ALTER COLUMN column_name [SET DEFAULT value | DROP DEFAULT]
            ALTER COLUMN column_name [SET NOT NULL| DROP NOT NULL]
        RENAME COLUMN column_name TO new_column_name;

    check:
        ADD CHECK (target IN ('_self', '_blank', '_parent', '_top'));
    tablename:
        RENAME TO new_table_name;
    constraint:
        ADD CONSTRAINT constraint_name constraint_definition

### default
To set a new default for a column, use a command like:

    ALTER TABLE products ALTER COLUMN price SET DEFAULT 7.77;

To remove any default value, use:

    ALTER TABLE products ALTER COLUMN price DROP DEFAULT;

### autoincrement
删除id, 再重建

    ALTER TABLE player drop column id;
    ALTER TABLE player ADD COLUMN id SERIAL PRIMARY KEY;

也可不删除id, 创建自增序列：

    create sequence player_id_seq;
    alter table player alter playerid set default nextval('player_id_seq');
    Select setval('player_id_seq', 2000051 ); --set to the highest current value of playerID