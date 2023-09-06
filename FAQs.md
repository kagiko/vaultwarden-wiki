# Frequently Asked Questions

## Is Vaultwarden associated with the Bitwarden project or Bitwarden, Inc?
Short answer, **No**.
There sometimes is some contact between the developers of both projects, but there is no collaboration.
Besides that, the Vaultwarden project only uses the web vault provided by Bitwarden, Inc with some [patches](https://github.com/dani-garcia/bw_web_builds/tree/master/patches) to make it work with our implementation.
<br>
<br>


## Can Vaultwarden connect to an Oracle MySQL V8.x database?
It could happen that you get the following warning when trying to start Vaultwarden when using Oracle MySQL v8.x
```
[vaultwarden::util][WARN] Can't connect to database, retrying: DieselConError.
[CAUSE] BadConnection(
    "Authentication plugin \'caching_sha2_password\' cannot be loaded: /usr/lib/x86_64-linux-gnu/mariadb18/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory",
)
```
Oracle MySQL v8.x uses a more secure password hashing method by default, which is good, but currently not supported by our builds.  
You need to create the Vaultwarden user in a specific way so that it uses the old native password hashing.
```sql
-- Use this on MySQLv8 installations
CREATE USER 'vaultwarden'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```
If you already created the user and only want to change the hashing method, use the following:
```sql
-- Change password type from caching_sha2_password to native
ALTER USER 'vaultwarden'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```
Also see: [Using MariaDB - Create Database and User](https://github.com/dani-garcia/vaultwarden/wiki/Using-the-MariaDB-(MySQL)-Backend#create-database-and-user)
<br>
<br>

## My client (Desktop, Mobile, Web) does not work, I can not login or it complains about invalid certificates.
The Bitwarden clients need a secure connection to fully work without any issues. Though some clients can work without a secure connection we do not recommend this.  
Most of the time when people are using certificates and still have issues, they are using so called Self-Signed certificates. While those could provide a secure connection, some platforms do not allow or support this.  
We recommend to use a service like Lets Encrypt to provide a valid and by most devices by default accepted certificate.  
See the following page:
* [Enabling-HTTPS](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-HTTPS)
* [Running a private vaultwarden instance with Let's Encrypt certs](https://github.com/dani-garcia/vaultwarden/wiki/Running-a-private-vaultwarden-instance-with-Let%27s-Encrypt-certs)
<br>

## Why do I see no icons for all my vault items?
There are multiple reasons why there is no icon shown.  
If it is just for a few of the vault items it could be that we are not able to extract it. Some sites have some protections enabled which causes our implementation to fail. Most of them require Javascript to work.  

It could also be that the Vaultwarden server is not able to access the Internet or resolve DNS queries.  
You could check the `/admin/diagnostics` page (See [[Enabling admin page]]) to see if you can resolve DNS queries and have a connection to the Internet.  
If that does work, there could also be a firewall or Outgoing Internet Proxy which maybe blocks these requests.
<br>
<br>

## Websocket connections show wrong IP address
This is not something we can fix on our side. The library we use doesn't support any form of `X-Forwarded-For` or `Forward` headers.  
It will always show the IP of the reverse proxy used unless you run vaultwarden directly without any proxy, or run a transparent proxy, that could cause it to do show the correct IP. It is not an important part to log, and if you use a reverse proxy you can probably also see this request in its logs which will have the correct IP.
<br>
<br>

## Why does Vaultwarden say `[INFO] No .env file found.` even though I provided one?
While launching, Vaultwarden checks for the existence of a file called `.env` (if not changed via the environment variable `ENV_FILE`) in the current working directory of the process. If you have not provided this file Vaultwarden will simply _inform_ you that it has not found it. This file is not related to the env file you have provided to docker which loads in the environment variables on container creation or the EnvironmentFile used within a [systemd .service](https://github.com/dani-garcia/vaultwarden/wiki/Setup-as-a-systemd-service).
<br>
<br>

## Can i run Vaultwarden as an Azure WebApp
Unfortunately Azure WebApp's uses CIFS/Samba as their volume storage which does not support locking. This causes issues with the SQLite database file.  
There are two ways to solve this.
1. Do not use SQLite, but MariaDB/MySQL or Posgresql as the database backend.
2. Try to disable WAL using the `ENABLE_DB_WAL` environment variable by setting it's value to `false`. This needs to be done on a new file, so you need to remove the previously created `db.sqlite3` file and restart the Vaultwarden app again.
<br>
<br>

## I did not find my answer here in the FAQ, what to do next?
Well, please try to search and click through our wonderful [Wiki](https://github.com/dani-garcia/vaultwarden/wiki).  
If that does not help you, try to look at either [Github Discussions](https://github.com/dani-garcia/vaultwarden/discussions) or [Vaultwarden Forums](https://vaultwarden.discourse.group/).  
If that also results in no solution, you could try to search the open and closed [Issues](https://github.com/dani-garcia/vaultwarden/issues).

If you still have not found an answer, you can start a topic on either [Github Discussions](https://github.com/dani-garcia/vaultwarden/discussions) or [Vaultwarden Forums](https://vaultwarden.discourse.group/), or joins us in our [chatroom](https://matrix.to/#/#vaultwarden:matrix.org).
