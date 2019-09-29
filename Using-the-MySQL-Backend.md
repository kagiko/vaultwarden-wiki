To use the MySQL backend, first ensure you build the binary [with MySQL feature enabled](https://github.com/dani-garcia/bitwarden_rs/wiki/Building-binary#mysql-backend).

To run the binary or container ensure the ```DATABASE_URL``` environment variable is set (i.e. ```DATABASE_URL='mysql://<user>:<password>@mysql/bitwarden```) and ```ENABLE_DB_WAL``` is set to false ```ENABLE_DB_WAL='false'``` .

**Connection String Syntax:**
```
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```
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
