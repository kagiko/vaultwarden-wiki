:warning: :poop: :warning:

Our builds are based upon MariaDB client libraries since that is what Debian provides.
We do not support the latest Oracle MySQLv8 version. If you insist to use MySQLv8 instead of MariaDB then create a user using an old password hashing method instead of the default one!

:warning: :poop: :warning:

:warning: Alpine is currently **not** support, on amd64 Alpine supports sqlite and postgresql, on armv7 it only supports sqlite.

To use the MariaDB (MySQL) backend, you can either use the [official Docker image](https://hub.docker.com/r/bitwardenrs/server) or build your own binary [with MySQL enabled](https://github.com/dani-garcia/bitwarden_rs/wiki/Building-binary#mysql-backend).

To run the binary or container, ensure the ```DATABASE_URL``` environment variable is set (i.e. ```DATABASE_URL='mysql://<user>:<password>@mysql/bitwarden'```).

**Connection String Syntax:**
```ini
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```
If your password contains special characters, you will need to use percentage encoding.

| ! | # | $ | % | & | ' | ( | ) | * | + | , | / | : | ; | = | ? | @ | [ | ] |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

A complete list of codes can be found on [Wikipedia page for percent encoding](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)

**Example using Docker:**
```bash
# Start a mysql container
docker run --name mysql --net <some-docker-network>\
 -e MYSQL_ROOT_PASSWORD=<my-secret-pw>\
 -e MYSQL_DATABASE=bitwarden\
 -e MYSQL_USER=<bitwarden_user>\
 -e MYSQL_PASSWORD=<bitwarden_pw> -d mysql:5.7

# Start bitwarden_rs with MySQL Env Vars set.
docker run -d --name bitwarden --net <some-docker-network>\
 -v $(pwd)/bw-data/:/data/ -v <Path to ssl certs>:/ssl/\
 -p 443:80 -e ROCKET_TLS='{certs="/ssl/<your ssl cert>",key="/ssl/<your ssl key>"}'\
 -e RUST_BACKTRACE=1 -e DATABASE_URL='mysql://<bitwarden_user>:<bitwarden_pw>@mysql/bitwarden'\
 -e ADMIN_TOKEN=<some_random_token_as_per_above_explanation>\
 -e ENABLE_DB_WAL='false' <you bitwarden_rs image name>
```

**Example using Non-Docker MySQL Server:**

```
Server IP/Port 192.168.1.10:3306 UN: dbuser / PW: yourpassword / DB: bitwarden
mysql://dbuser:yourpassword@192.168.1.10:3306/bitwarden
```

**Example using docker-compose

```yaml
version: "3.7"
services:
 mariadb:
  image: "mariadb"
  container_name: "mariadb"
  hostname: "mariadb"
  restart: always
  env_file:
   - ".env"
  volumes:
   - "mariadb_vol:/var/lib/mysql"
   - "/etc/localtime:/etc/localtime:ro"
  environment:
   - "MYSQL_ROOT_PASSWORD=<my-secret-pw>"
   - "MYSQL_PASSWORD=<bitwarden_pw>"
   - "MYSQL_DATABASE=bitwarden_db"
   - "MYSQL_USER=<bitwarden_user>"

 bitwarden:
  image: "bitwardenrs/server-mysql:latest"
  container_name: "bitwarden"
  hostname: "bitwarden"
  restart: always
  env_file:
   - ".env"
  volumes:
   - "bitwarden_vol:/data/"
  environment:
## Had issues when using single parentheses around the mysql URL as in the plain docker example 
   - "DATABASE_URL=mysql://<bitwarden_user>:<bitwarden_pw>@mariadb/bitwarden_db"
   - "ADMIN_TOKEN=<some_random_token_as_per_above_explanation>"
   - "RUST_BACKTRACE=1"
  ports:
   - "80:80"

volumes:
 bitwarden_vol:
 mariadb_vol:
```

**Migrating from SQLite to MySQL**

An easy way of migrating from SQLite to MySQL has been described in this [issue comment](https://github.com/dani-garcia/bitwarden_rs/issues/497#issuecomment-511827057). The steps are repeated below. Please, note that you are using this at your own risk and you are strongly advised to backup your installation and data!

1. Create an new (empty) database for bitwarden_rs:
```sql
CREATE DATABASE bitwarden_rs;
```
2. Create a new database user and grant rights to database:
```sql
CREATE USER 'bitwarden_rs'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL ON `bitwarden_rs`.* TO 'bitwarden_rs'@'localhost';
FLUSH PRIVILEGES;
```
You might want to try a restricted set of grants:
```sql
CREATE USER 'bitwarden_rs'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALTER, CREATE, DELETE, DROP, INDEX, INSERT, SELECT, UPDATE ON `bitwarden_rs`.* TO 'bitwarden_rs'@'localhost';
FLUSH PRIVILEGES;
```
3. Configure bitwarden_rs and start it, so diesel can run migrations and set up the schema properly. Do not do anything else.
4. Stop bitwarden_rs.
5. Dump your existing SQLite database using the following command. Double check the name of your sqlite database, default should be db.sqlite.<br>
**Note:** You need the sqlite3 command installed on your Linux system.<br>
We need to remove some queries from the output of the sqlite dump like create table etc.. we will do that here.<br><br>
You either can use this one-liner:
```bash
sqlite3 db.sqlite3 .dump | grep "^INSERT INTO" | grep -v "__diesel_schema_migrations" > sqlitedump.sql ; echo -ne "SET FOREIGN_KEY_CHECKS=0;\n$(cat sqlitedump.sql)" > mysqldump.sql
```
Or the following right after each other:
```bash
sqlite3 db.sqlite3 .dump | grep "^INSERT INTO" | grep -v "__diesel_schema_migrations" > sqlitedump.sql
echo "SET FOREIGN_KEY_CHECKS=0;" > mysqldump.sql
cat sqlitedump.sql >> mysqldump.sql
```
6. Load your MySQL dump:
```bash
mysql --force --password --user=bitwarden_rs --database=bitwarden_rs < mysqldump.sql
```
7. Start bitwarden_rs again.

*Note: Loading your MySQL dump with ```--show-warnings``` will highlight that the datetime fields are getting truncated during the import which **seems** to be okay.*
```
Note (Code 1265): Data truncated for column 'created_at' at row 1
Note (Code 1265): Data truncated for column 'updated_at' at row 1
```

*Note1:Then error loading data  mysqldump.sql Load error*
```
error (1064): Syntax error near '"users" VALUES('9b5c2d13-8c4f-47e9-bd94-f0d7036ff581'*********)
```
fix:
```bash
sed -i 's#\"#\#g' mysqldump.sql
```
```bash
mysql --password --user=bitwarden_rs
use bitwarden_rs
source /bw-data/mysqldump.sql
exit
```
