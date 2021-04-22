# FAQs

## Is Bitwarden_RS associated with the Bitwarden project or 8bit Solutions LLC?
Short answer, **No**.
There sometimes is some contact between the developers of both projects, but there is no collaboration.
Besides that, the Bitwarden_RS project only uses the web vault provided by 8bit Solutions LLC with some patches to make it work with our implementation.
<br>
<br>

## Can Bitwarden_RS connect to an Oracle MySQL V8.x database?
It could happen that you get the following warning when trying to start Bitwarden_RS when using Oracle MySQL v8.x
```
[bitwarden_rs::util][WARN] Can't connect to database, retrying: DieselConError.
[CAUSE] BadConnection(
    "Authentication plugin \'caching_sha2_password\' cannot be loaded: /usr/lib/x86_64-linux-gnu/mariadb18/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory",
)
```
Oracle MySQL v8.x uses a more secure password hashing method by default, which is good, but currently not supported by our builds.  
You need to create the Bitwarden_RS user in a specific way so that it uses the old native password hashing.
```sql
-- Use this on MySQLv8 installations
CREATE USER 'bitwarden_rs'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```
If you already created the user and only want to change the hashing method, use the following:
```sql
-- Change password type from caching_sha2_password to native
ALTER USER 'bitwarden_rs'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
```
Also see: [Using MariaDB - Create Database and User](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-the-MariaDB-(MySQL)-Backend#create-database-and-user)
<br>
<br>

## My client (Desktop, Mobile, Web) does not work, I can not login or it complains about invalid certificates.
The Bitwarden clients need a secure connection to fully work without any issues. Though some clients can work without a secure connection we do not recommend this.  
Most of the time when people are using certificates and still have issues, they are using so called Self-Signed certificates. While those could provide a secure connection, some platforms do not allow or support this.  
We recommend to use a service like Lets Encrypt to provide a valid and by most devices by default accepted certificate.  
See the following page:
* [Enabling-HTTPS](https://github.com/dani-garcia/bitwarden_rs/wiki/Enabling-HTTPS)
* [Running a private bitwarden_rs instance with Let's Encrypt certs](https://github.com/dani-garcia/bitwarden_rs/wiki/Running-a-private-bitwarden_rs-instance-with-Let%27s-Encrypt-certs)
<br>
<br>

## Why do I see no icons for all my vault items?
There are multiple reasons why there is no icon shown.  
If it is just for a few of the vault items it could be that we are not able to extract it. Some sites have some protections enabled which causes our implementation to fail. Most of them require Javascript to work.  

It could also be that the Bitwarden_RS server is not able to access the Internet or resolve DNS queries.  
You could check the `/admin/diagnostics` page to see if you can resolve DNS queries and have a connection to the Internet.  
If that does work, there could also be a firewall or Outgoing Internet Proxy which maybe blocks these requests.
<br>
<br>

## Can i run Bitwarden_RS as an Azure WebApp
Unfortunately Azure WebApp's uses CIFS/Samba as there volume storage which does not support locking. This causes issues with the SQLite database file.  
There are two ways to solve this.
1. Do not use SQLite, but MariaDB/MySQL or Posgresql as the database backend.
2. Try to disable WAL using the `ENABLE_DB_WAL` environment variable by setting it's value to `false`. This needs to be done on a new file, so you need to remove the previously created `db.sqlite3` file and restart the Bitwarden_RS app again.
<br>
<br>

## I did not find my answer here in the FAQ, what to do next?
Well, please try to search and click through our wonderful [Wiki](https://github.com/dani-garcia/bitwarden_rs/wiki).  
If that does not help you, try to look at either [Github Discussions](https://github.com/dani-garcia/bitwarden_rs/discussions) or [Bitwarden_RS Forums](https://bitwardenrs.discourse.group/).  
If that also results in no solution, you could try to search the open and closed [Issues](https://github.com/dani-garcia/bitwarden_rs/issues).

If you still have not found an answer, you can start a topic on either [Github Discussions](https://github.com/dani-garcia/bitwarden_rs/discussions) or [Bitwarden_RS Forums](https://bitwardenrs.discourse.group/), or joins us in our [chatroom](https://matrix.to/#/#bitwarden_rs:matrix.org).
