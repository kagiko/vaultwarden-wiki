> [!WARNING]
> <p align="center">⚠️ ☠️ ⚠️</p>
> 
> <p align="center"><b>Use these commands at your own risk!<br>Always create a backup before you do anything which could destroy your whole vault!</b></p>
> 
> <p align="center">⚠️ ☠️ ⚠️</p>

---

## General
Vaultwarden was designed at first using SQLite only, but at the time both MariaDB (MySQL) and PostgreSQL also were added into the mix.
For SQLite you do not have to run a separate server or container, while for the other two you do need to run something extra.

Now, what if you started out using MariaDB and want to go back to SQLite?
Well, that is possible, but there could be some quirks which we do not know of using the steps below. If you encounter anything strange with this and need help, or even solved it, please open a new discussion here: https://github.com/dani-garcia/vaultwarden/discussions to help you and others.

# How to migrate from MariaDB to SQLite

Make sure you are using the same version of Vaultwarden (Docker or custom build) for both SQLite and MariaDB, don't update the Docker image in-between these steps.
To migrate to SQLite we first need to have a SQLite database file which we can use to transfer the data to.
For this file to be created you need to stop your current Vaultwarden instance, and configure it to use SQLite.
You can do this by changing your `DATABASE_URL` from `DATABASE_URL=mysql://<vaultwarden_user>:<vaultwarden_pw>@mariadb/vaultwarden` to `DATABASE_URL=/data/db.sqlite3` for example. ( `/data` is the internal path within the Docker container of the `-v` volume you use).

After you have changed the config, start Vaultwarden and it should show you that it executed a few migrations by checking the logs for lines which start with `Executing migration script ....`.

Now stop Vaultwarden again so that you can start the migration.
You need the database host and credentials you used for MariaDB to continue.

Now run the following one-liner and adjust the `<dbhost>`, `<dbuser>` and `<database>` to what you used for your MariaDB connection.

```bash
mysqldump \
  --host=<dbhost> \
  --user=<dbuser> --password \
  --skip-create-options \
  --compatible=ansi \
  --skip-extended-insert \
  --compact \
  --single-transaction \
  --no-create-db \
  --no-create-info \
  --hex-blob <database> \
  | grep -a "^INSERT INTO" | grep -a -v "__diesel_schema_migrations" \
  | sed 's#\\"#"#gm' \
  | sed -sE "s#,0x([^,]*)#,X'\L\1'#gm" \
   > mysql-to-sqlite.sql
```

You should be prompted to enter a password, enter your password and press enter.

This should generate a file `mysql-to-sqlite.sql` which holds your database.
Now look for the db.sqlite3 file Vaultwarden created in the previous step when you started Vaultwarden for the first time using SQLite as it's database.
Copy or move `mysql-to-sqlite.sql` to the same directory as `db.sqlite3`.
Now you can execute the following

```bash
sqlite3 db.sqlite3 < mysql-to-sqlite.sql
```

This should have filled the SQLite database with the dump, and you can now start Vaultwarden again using SQLite instead of MySQL.
