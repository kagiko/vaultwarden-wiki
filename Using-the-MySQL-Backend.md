To use the MySQL backend, first ensure you build the binary [with MySQL feature enabled](https://github.com/dani-garcia/bitwarden_rs/wiki/Building-binary#mysql-backend).

To run the binary or container ensure the ```DATABASE_URL``` environment variable is set (i.e. ```DATABASE_URL='mysql://<user>:<password>@mysql/bitwarden```) and ```ENABLE_DB_WAL``` is set to false ```ENABLE_DB_WAL='false'``` .

**Connection String Syntax:**
```
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```
If your password contains special characters, you will need to use percentage encoding.

| ! | # | $ | % | & | ' | ( | ) | * | + | , | / | : | ; | = | ? | @ | [ | ] |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| %21 | %23 | %24 | %25 | %26 | %27 | %28 | %29 | %2A | %2B | %2C | %2F | %3A | %3B | %3D | %3F | %40 | %5B | %5D |

A complete list of codes can be found on [Wikipedia page for percent encoding](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters)

**Example using Docker:**
```
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
Server IP/Port 192.168.1.10:3306 UN: dbuser / PW: P@ssw0rd / DB: Bitwarden
mysql://dbuser:P@ssw0rd@192.168.1.10:3306/bitwarden
```

**Migrating from SQLite to MySQL**

An easy way of migrating from SQLite to MySQL has been described in this [issue comment](https://github.com/dani-garcia/bitwarden_rs/issues/497#issuecomment-511827057). The steps are repeated below. Please, note that you are using this at your won risk and you are strongly advised to backup your installation and data!

1. Create an new (empty) database for bitwarden_rs:
```CREATE DATABASE bitwarden_rs;```
2. Create a new database user and grant rights to database:
```
CREATE USER 'bitwarden_rs'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL ON `bitwarden_rs`.* TO 'bitwarden_rs'@'localhost';
FLUSH PRIVILEGES;
```
3. Configure bitwarden_rs and start it, so diesel can run migrations and set up the schema properly. Do not do anything else.
4. Stop bitwarden_rs.
5. Dump your existing SQLite database: ```sqlite3 db.sqlite3 .dump > sqlitedump.sql```
NB: On Debian (Buster), you'll need to install sqlite3 for this
6. Drop schema creation and diesel metadata from your dump, leaving only your actual data: ```grep "INSERT INTO" sqlitedump.sql | grep -v "__diesel_schema_migrations" > mysqldump.sql```
7. Load your MySQL dump: ```mysql -ubitwarden_rs -pyourpassword < mysqldump.sql```
NB: You might want to use ```--show-warnings```
8. Start bitwarden_rs.
