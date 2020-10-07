To use the PostgreSQL backend, you can either use the [official Docker image](https://hub.docker.com/r/bitwardenrs/server-postgresql) or build your own binary [with PostgreSQL enabled](https://github.com/dani-garcia/bitwarden_rs/wiki/Building-binary#postgresql-backend). The official Docker image is currently only available for x86_64 due to cross-compilation issues, so if you're using another platform, you'll need to build your own binary or use a third-party binary or Docker image.

To run the binary or container ensure the ```DATABASE_URL``` environment variable is set (i.e. ```DATABASE_URL='mysql://<user>:<password>@mysql/bitwarden'```) and ```ENABLE_DB_WAL``` is set to false ```ENABLE_DB_WAL='false'``` .

**Connection String Syntax:**
```ini
DATABASE_URL=postgresql://[[user]:[password]@]host[:port][/database]
```
If your password contains special characters, you will need to use percentage encoding.

| ! | # | $ | % | & | ' | ( | ) | * | + | , | / | : | ; | = | ? | @ | [ | ] |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

A complete list of codes can be found on [Wikipedia page for percent encoding](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)

**Migrating from SQLite to PostgreSQL**

An easy way of migrating from SQLite to PostgreSQL or to MySQL exists, but please, note that you **are using this at your own risk and you are strongly advised to backup your installation and data!**. This is **unsupported** and has not been robustly tested.

1. Create an new (empty) database for bitwarden_rs:
```sql
CREATE DATABASE bitwarden_rs;
```
2. Create a new database user and grant rights to database:
```sql
CREATE USER bitwarden_rs WITH ENCRYPTED PASSWORD 'yourpassword';
GRANT all privileges ON database bitwarden_rs TO bitwarden_rs;
```
3. Configure bitwarden_rs and start it, so diesel can run migrations and set up the schema properly. Do not do anything else.
4. Stop bitwarden_rs.
5. install [pgloader](http://pgloader.io/)
6. create the file bitwarden.load with the following content:
```
load database
     from sqlite://yoursqliteuser:yoursqlitepassword@yoursqliteserver:yoursqliteport/yoursqlitedatabase
     into postgresql://yourpgsqluser:yourpgsqlpassword:yourpgsqlserver:yourpgsqlport/yourpgsqldatabase
     WITH data only, include no drop, reset sequences
     EXCLUDING TABLE NAMES MATCHING '__diesel_schema_migrations'
     ALTER SCHEMA 'bitwarden' RENAME TO 'public'
;
```
7. run the command ```pgloader bitwarden.load``` and you might see some warnings, but the migration should complete successfully
8. Start bitwarden_rs again.
