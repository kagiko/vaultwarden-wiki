To use the PostgreSQL backend, you can either use the [official Docker image](https://hub.docker.com/r/vaultwarden/server) or build your own binary [with PostgreSQL enabled](https://github.com/dani-garcia/vaultwarden/wiki/Building-binary#postgresql-backend).

To run the binary or container ensure the `DATABASE_URL` environment variable is set (i.e. `DATABASE_URL='postgresql://<user>:<password>@postgresql/bitwarden'`)

**Connection String Syntax:**
```ini
DATABASE_URL=postgresql://[[user]:[password]@]host[:port][/database]
```
An example docker run environment variable would be: ```-e 'DATABASE_URL=postgresql://user_name:user_password@db_host:5432/vaultwarden'```.

If you need to set additional connection parameters, note that the `DATABASE_URL` value ends up getting parsed by `libpq`, so you can use any of the parameters listed in the `libpq` [docs](https://www.postgresql.org/docs/current/libpq-envars.html). You can either add the connection parameter to `DATABASE_URL` or specify it via its corresponding `PG*` environment variable. If running under Docker, keep in mind that any paths provided need to be from the perspective of the Docker container, not the Docker host.

If you want to use a custom schema/search-path you need to use the following connection string:<br>
Note the URL-encoded characters such as `%20` for the space and `%3D` for `=` sign
```ini
DATABASE_URL=postgresql://user_name:user_password@db_host:5432/vaultwarden?application_name=vaultwarden&options=-c%20search_path%3Ddb_schema
```

If your password contains special characters, you will need to use percentage encoding.

| ! | # | $ | % | & | ' | ( | ) | * | + | , | / | : | ; | = | ? | @ | [ | ] |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

A complete list of codes can be found on [Wikipedia page for percent encoding](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)

## Migrating from SQLite to PostgreSQL

An easy way of migrating from SQLite to PostgreSQL or to MySQL exists, but please, note that you **are using this at your own risk and you are strongly advised to backup your installation and data!**. This is **unsupported** and has not been robustly tested.

1. Create an new (empty) database for vaultwarden:
```sql
CREATE DATABASE vaultwarden;
```
2. Create a new database user and grant rights to database:
```sql
CREATE USER vaultwarden WITH ENCRYPTED PASSWORD 'yourpassword';
GRANT ALL ON DATABASE vaultwarden TO vaultwarden;
GRANT all privileges ON database vaultwarden TO vaultwarden;
```
3. Configure vaultwarden and start it, so diesel can run migrations and set up the schema properly. Do not do anything else.
4. Stop vaultwarden.
5. install [pgloader](http://pgloader.io/)
6. [disable WAL](https://github.com/dani-garcia/vaultwarden/wiki/Running-without-WAL-enabled#1-disable-wal-on-old-db) of the SQLite database.
7. create the file bitwarden.load with the following content:
```
load database
     from sqlite:///where/you/keep/your/vaultwarden/db.sqlite3 
     into postgresql://yourpgsqluser:yourpgsqlpassword@yourpgsqlserver:yourpgsqlport/yourpgsqldatabase
     WITH data only, include no drop, reset sequences
     EXCLUDING TABLE NAMES LIKE '__diesel_schema_migrations'
     ALTER SCHEMA 'bitwarden' RENAME TO 'public'
;
```
8. run the command ```pgloader bitwarden.load``` and you might see some warnings, but the migration should complete successfully
9. Start vaultwarden again.

## Migrating from MySQL to PostgreSQL
> Tested with MariaDB 10.3.32, PostgreSQL 14.2 and Vaultwarden 1.24.0


Please, note that you **are using this at your own risk and you are strongly advised to backup your installation and data!**. This is **unsupported** and has not been robustly tested.

1. Create a new (empty) database for vaultwarden:
```sql
CREATE DATABASE vaultwarden;
```
2. Create a new database user and grant rights to database:
```sql
CREATE USER vaultwarden WITH ENCRYPTED PASSWORD 'yourpassword';
GRANT ALL ON DATABASE vaultwarden TO vaultwarden;
GRANT all privileges ON database vaultwarden TO vaultwarden;
```
3. Configure vaultwarden and start it, so diesel can run migrations and set up the schema properly. Do not do anything else.
4. Stop vaultwarden.
5. Install [pgloader](http://pgloader.io/). Make sure that you have the latest version of pgloader, the official Ubuntu repository has an outdated version which does not work well with newer versions of PostgreSQL. The newest version can be obtained from the [PostgreSQL Apt Repository](https://www.postgresql.org/download/linux/ubuntu/)
6. Create the file `vaultwarden.load ` with the following content:
```
load database
     from mysql://yourmysqluser:yourmysqlpassword@yourmysqlserver:yourmysqlport/yourmysqldatabase 
     into postgresql://yourpgsqluser:yourpgsqlpassword@yourpgsqlserver:yourpgsqlport/yourpgsqldatabase
     WITH data only
     EXCLUDING TABLE NAMES MATCHING '__diesel_schema_migrations'
     ALTER SCHEMA 'vaultwarden' RENAME TO 'public'
;
```
_Optionally add `?sslmode=require` to the PostgreSQL connection string if your connection requires SSL_

7. Run the command ```pgloader vaultwarden.load```  and you might see some warnings, but the migration should complete successfully. If there are errors, it is likely that you have an outdated version of pgloader!
8. Start vaultwarden again