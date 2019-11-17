To use the MySQL backend, first ensure you build the binary [with MySQL feature enabled](https://github.com/dani-garcia/bitwarden_rs/wiki/Building-binary#mysql-backend).

To run the binary or container ensure the ```DATABASE_URL``` environment variable is set (i.e. ```DATABASE_URL='mysql://<user>:<password>@mysql/bitwarden```) and ```ENABLE_DB_WAL``` is set to false ```ENABLE_DB_WAL='false'``` .

**Connection String Syntax:**
```
DATABASE_URL=mysql://[[user]:[password]@]host[:port][/database]
```
If your password contains special characters, you will need to use percentage encoding.

{| cellpadding="6px" border=1 style="border:1px solid #C0C0C0; border-collapse:collapse; background-color:white;" class="wikitable"
|+Reserved characters after percent-encoding
|-
| <code>[[exclamation mark|!]]</code> || <code>[[number sign|#]]</code> || <code>[[dollar sign|$]]</code> || <code>[[Percent sign|%]]</code> || <code>[[ampersand|&]]</code> || <code>[[apostrophe (mark)|']]</code> || <code>[[parenthesis|(]]</code> || <code>[[parenthesis|)]]</code> || <code>[[asterisk|<nowiki>*</nowiki>]]</code> || <code>[[plus sign|+]]</code> || <code>[[Comma|,]]</code> || <code>[[slash (punctuation)|/]]</code> || <code>[[colon (punctuation)|:]]</code> || <code>[[semicolon|;]]</code> || <code>[[equal sign|=]]</code> || <code>[[question mark|?]]</code> || <code>[[@]]</code> || <code>[[square_bracket|[]]</code> || <code>[[square_bracket|<nowiki>]</nowiki>]]</code>
|-
| <code>%21</code> || <code>%23</code> || <code>%24</code> || <code>%25</code> || <code>%26</code> || <code>%27</code> || <code>%28</code> || <code>%29</code> || <code>%2A</code> || <code>%2B</code> || <code>%2C</code> || <code>%2F</code> || <code>%3A</code> || <code>%3B</code> || <code>%3D</code>|| <code>%3F</code> || <code>%40</code> || <code>%5B</code> || <code>%5D</code>
|}

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
