> [!WARNING]
> <p align=center>⚠️ 💩 ⚠️</p>
>
> <p align=center>Although MySQL database works fine, be aware that our builds are based upon MariaDB client libraries since that is what Debian provides.</p>
>
> <p align=center>⚠️ 💩 ⚠️</p>


To use the MariaDB (MySQL) backend, you can either use the [official Docker image](https://hub.docker.com/r/vaultwarden/server) or build your own binary [with MySQL enabled](https://github.com/dani-garcia/vaultwarden/wiki/Building-binary#mysql-backend).

To run the binary or container, ensure the ```DATABASE_URL``` environment variable is set (i.e. ```DATABASE_URL='mysql://<user>:<password>@mysql/vaultwarden[?ssl_mode=(disabled|required|preferred)&ssl_ca=/path/to/cart.(crt|pem)]'```).

**Connection String Syntax:**
```ini
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```
If your password contains special characters, you will need to use percentage encoding.

| ! | # | $ | % | & | ' | ( | ) | * | + | , | / | : | ; | = | ? | @ | [ | ] |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

A complete list of codes can be found on [Wikipedia page for percent encoding](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)

## Example using Docker

```bash
# Start a mysql container
docker run --name mysql --net <some-docker-network>\
 -e MYSQL_ROOT_PASSWORD=<my-secret-pw>\
 -e MYSQL_DATABASE=vaultwarden\
 -e MYSQL_USER=<vaultwarden_user>\
 -e MYSQL_PASSWORD=<vaultwarden_pw> -d mysql:5.7

# Start vaultwarden with MySQL Env Vars set.
docker run -d --name vaultwarden --net <some-docker-network>\
 -v $(pwd)/vw-data/:/data/ -v <Path to ssl certs>:/ssl/\
 -p 443:80 -e ROCKET_TLS='{certs="/ssl/<your ssl cert>",key="/ssl/<your ssl key>"}'\
 -e RUST_BACKTRACE=1 -e DATABASE_URL='mysql://<vaultwarden_user>:<vaultwarden_pw>@mysql/vaultwarden'\
 -e ADMIN_TOKEN=<some_random_token_as_per_above_explanation>\
 -e ENABLE_DB_WAL='false' <you vaultwarden image name>
```

### Example using Non-Docker MySQL Server

```
Server IP/Port 192.168.1.10:3306 UN: dbuser / PW: yourpassword / DB: vaultwarden
mysql://dbuser:yourpassword@192.168.1.10:3306/vaultwarden
```

### Example using docker-compose

```yaml
version: "3.7"
services:
 vaultwarden-db:
  image: "mariadb" # or "mysql"
  container_name: "vaultwarden-db"
  restart: always
  env_file:
   - ".env"
  volumes:
   - "vaultwarden-db_vol:/var/lib/mysql"
   - "/etc/localtime:/etc/localtime:ro"
  environment:
   - "MYSQL_ROOT_PASSWORD=<my-secret-pw>"
   - "MYSQL_PASSWORD=<vaultwarden_pw>"
   - "MYSQL_DATABASE=vaultwarden"
   - "MYSQL_USER=<vaultwarden_user>"
  healthcheck:
   test: mariadb-admin ping -h 127.0.0.1 -u $$MYSQL_USER --password=$$MYSQL_PASSWORD
   start_period: 5s
   interval: 5s
   timeout: 5s
   retries: 55


 vaultwarden:
  image: "vaultwarden/server:latest"
  container_name: "vaultwarden"
  hostname: "vaultwarden"
  depends_on:
   vaultwarden-db:
    condition: service_healthy
  restart: always
  env_file:
   - ".env"
  volumes:
   - "vaultwarden_vol:/data/"
  environment:
   - DATABASE_URL=mysql://<vaultwarden_user>:${VAULTWARDEN_MYSQL_PASSWORD}@vaultwarden-db/vaultwarden
   - ADMIN_TOKEN=<some_random_token_as_per_above_explanation> # https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page
   - RUST_BACKTRACE=1
  ports:
   - "80:80"

volumes:
 vaultwarden_vol:
 vaultwarden-db_vol:
```

<br>

## Manually create a database (For example, using an existing database server)

> [!WARNING]
> To execute these queries you need a user which has the privileges to create new databases and users.  
> Most of the time that would be the `root` user, but it could be different for your database.

>Using the docker-compose example above makes these steps unnecessary. Database, collation and charset is created automatically at startup.

### Create database and user

1. Create an new (empty) database for vaultwarden (Ensure the Charset and Collate are correct!):
```sql
CREATE DATABASE vaultwarden CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

2. Create a new database user and grant rights to database (MariaDB, MySQL):
```sql
CREATE USER 'vaultwarden'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL ON `vaultwarden`.* TO 'vaultwarden'@'localhost';
FLUSH PRIVILEGES;
```

You might want to try a restricted set of grants:
```sql
GRANT ALTER, CREATE, DELETE, DROP, INDEX, INSERT, REFERENCES, SELECT, UPDATE ON `vaultwarden`.* TO 'vaultwarden'@'localhost';
FLUSH PRIVILEGES;
```

<br>

## Migrating from SQLite to MySQL

An easy way of migrating from SQLite to MySQL has been described in this [issue comment](https://github.com/dani-garcia/vaultwarden/issues/497#issuecomment-511827057). The steps are repeated below. Please, note that you are using this at your own risk and you are strongly advised to backup your installation and data!

1. First follow the steps 1 and 2 above
2. Configure vaultwarden and start it, so diesel can run migrations and set up the schema properly. Do not do anything else.
3. Stop vaultwarden.
4. Dump your existing SQLite database using the following command. Double check the name of your sqlite database, default should be db.sqlite.<br>
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
5. Load your MySQL dump:
```bash
mysql --force --password --user=vaultwarden --database=vaultwarden < mysqldump.sql
```
6. Start vaultwarden again.

*Note: Loading your MySQL dump with ```--show-warnings``` will highlight that the datetime fields are getting truncated during the import which **seems** to be okay.*
```
Note (Code 1265): Data truncated for column 'created_at' at row 1
Note (Code 1265): Data truncated for column 'updated_at' at row 1
```

*Note 1:Then error loading data  mysqldump.sql Load error*
```
error (1064): Syntax error near '"users" VALUES('9b5c2d13-8c4f-47e9-bd94-f0d7036ff581'*********)
```
fix:
```bash
sed -i 's#\"#\#g' mysqldump.sql
```
```bash
mysql --password --user=vaultwarden
use vaultwarden
source /vw-data/mysqldump.sql
exit
```

*Note 2: If MariaDB might complain about mismatched value counts if SQLite database is migrated from a prior, older version, e.g.:*
```
ERROR 1136 (21S01) at line ###: Column count doesn't match value count at row 1
```
The version jump may have added new database columns. Upgrade vaultwarden using the SQLite backend first to run migrations on the SQLite database, switch to the MariaDB backend, then repeat migration steps above. Alternatively, look for the commits adding migrations since the version you had installed and run migrations manually using `sqlite3`

<br>
<br>

## Foreign key errors, collation and charset

Because some data stored within the vault is either binary, or plain-text like mail addresses, user names or organization names which can contain Unicode characters you need to make sure the collation and charset of your database and tables is correctly set. If this is not the case, it could cause issues during updates which then generates messages like `Cannot add or update a child row: a foreign key constraint fails ...` or `Row size too large. The maximum row size for the used table type, not counting BLOBs, is 8126.`.

To solve this you need to update/change the collation and charset of both the whole database and the tables it contains.
You can do that by following and executing the following sets via your perfered SQL tool, or using the CLI.

In the examples below i will use the database name `vaultwarden`, change this if you used a different name.

<br>

Before starting, verify if there are any issue by running the following two queries.<br>
It should return `utf8mb4` and `utf8mb4_unicode_ci`.<br>
Also run these queries at the end of the queries below to verify it worked!

```mysql
SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME FROM information_schema.SCHEMATA WHERE SCHEMA_NAME = "vaultwarden";
SELECT CHARACTER_SET_NAME, COLLATION_NAME FROM information_schema.`COLUMNS` WHERE TABLE_SCHEMA = "vaultwarden" AND CHARACTER_SET_NAME IS NOT NULL;
```

<br>

First change the collation and charset of the database it self:
```mysql
ALTER DATABASE `vaultwarden` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

After that convert all the tables (including the text fields).
Execute the following, and copy the output.
```mysql
SELECT CONCAT('ALTER TABLE `', TABLE_NAME,'` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;') AS CharSetConvert
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA="vaultwarden"
AND TABLE_TYPE="BASE TABLE";
```

This will generate several queries which you need to execute to convert both the collation and the charset of these tables.
For these changes to work we need to temporarily disable foreign key checking.
Copy/Paste the output from the generated output from the query above between the following lines:
```mysql
SET foreign_key_checks=0;
-- Copy/Paste the output from above here
SET foreign_key_checks=1;
```

In the end it should look something like the following output (but it could be different depending on updates or changes to the database structure).:
```mysql
SET foreign_key_checks=0;
ALTER TABLE `__diesel_schema_migrations` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `attachments` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `ciphers_collections` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `ciphers` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `collections` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `devices` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `emergency_access` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `favorites` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `folders_ciphers` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `folders` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `invitations` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `org_policies` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `organizations` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `sends` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `twofactor_incomplete` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `twofactor` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `users_collections` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `users_organizations` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE `users` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
SET foreign_key_checks=1;
```

You need to run those queries to convert them to the correct collation and charset.
You can verify if it worked by running the following query on at least one table.
```mysql
SHOW CREATE TABLE `users`; 
```

It should output something like this, note the `CHARSET=utf8mb4` at the end.
```mysql
CREATE TABLE `users` (
  `uuid` char(36) NOT NULL,
  `created_at` datetime NOT NULL,
  `updated_at` datetime NOT NULL,
  `email` varchar(255) NOT NULL,
  `name` text NOT NULL,
  --- CUT ---
  `enabled` tinyint(1) NOT NULL DEFAULT 1,
  `stamp_exception` text DEFAULT NULL,
  PRIMARY KEY (`uuid`),
  UNIQUE KEY `email` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

You can do the same for the database:
```mysql
SHOW CREATE DATABASE `vaultwarden`;
```

It should look something like this, note the `DEFAULT CHARACTER SET utf8mb4`.
```mysql
CREATE DATABASE `vaultwarden` /*!40100 DEFAULT CHARACTER SET utf8mb4 */
```
