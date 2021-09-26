To use the PostgreSQL backend, you can either use the [official Docker image](https://hub.docker.com/r/vaultwarden/server) or build your own binary [with PostgreSQL enabled](https://github.com/dani-garcia/vaultwarden/wiki/Building-binary#postgresql-backend).

To run the binary or container ensure the `DATABASE_URL` environment variable is set (i.e. `DATABASE_URL='postgresql://<user>:<password>@postgresql/bitwarden'`)

**Connection String Syntax:**
```ini
DATABASE_URL=postgresql://[[user]:[password]@]host[:port][/database]
```
An example docker run environment variable would be: ```-e 'DATABASE_URL=postgresql://postgresadmin:strongpassword@postgres:5432/vaultwarden'```.

If your password contains special characters, you will need to use percentage encoding.

| ! | # | $ | % | & | ' | ( | ) | * | + | , | / | : | ; | = | ? | @ | [ | ] |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

A complete list of codes can be found on [Wikipedia page for percent encoding](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)

**Migrating from SQLite to PostgreSQL**

An easy way of migrating from SQLite to PostgreSQL or to MySQL exists, but please, note that you **are using this at your own risk and you are strongly advised to backup your installation and data!**. This is **unsupported** and has not been robustly tested.

1. Create an new (empty) database for vaultwarden:
```sql
CREATE DATABASE vaultwarden;
```
2. Create a new database user and grant rights to database:
```sql
CREATE USER vaultwarden WITH ENCRYPTED PASSWORD 'yourpassword';
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
