As of release 1.5.0, bitwarden_rs supports logging to file. See [[Logging|logging]] for information on how to set this up.

## Logging Failed Login Attempts

After specifying the log file location, failed login attempts will appear in the logs in the following format:

```
[YYYY-MM-DD hh:mm:ss][bitwarden_rs::api::identity][ERROR] Username or password is incorrect. Try again. IP: XXX.XXX.XXX.XXX. Username: email@domain.com.
```
Install Fail2Ban (under Debian/Rasbian)
```
sudo apt-get install fail2ban -y
```
Under Fedora/CentOS

EPEL repository necessary (CentOS 7)
```
sudo yum install epel-release
```

```
sudo yum install fail2ban -y
```
## Fail2Ban Filter

Create the filter file
```
sudo nano /etc/fail2ban/filter.d/bitwarden.conf
```
And add the following
```
[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$
ignoreregex =
```

Use ```<HOST>``` instead of ```<ADDR>``` if you get the following error message in fail2ban.log
(CentOS 7, Fail2Ban v0.9.7):
```
fail2ban.filter         [5291]: ERROR   No 'host' group in '^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$'
```

## Fail2Ban Jail

Now we need the jail, create the jail file
```
sudo nano /etc/fail2ban/jail.d/bitwarden.local
```
and add:
```
[bitwarden]
enabled = true
port = 80,443,8081
filter = bitwarden
action = iptables-allports[name=bitwarden]
logpath = /path/to/bitwarden/log
maxretry = 3
bantime = 14400
findtime = 14400
```
Note: Docker uses the FORWARD chain instead of the default INPUT chain. Therefore use the following action when using Docker:

```
action = iptables-allports[name=bitwarden, chain=FORWARD]
```
**NOTE**: 
Do not use this if you use a reverse proxy before docker container. If proxy, like apache2 or nginx is used, use the ports of the proxy and do not use chain=FORWARD, only when using Docker **without** proxy!

**NOTE on the NOTE above**:
Thats at least not true for running on Docker (CentOS 7) with caddy as reverse proxy. chain=FORWARD is absolutely fine and working with caddy as reverse proxy.

Feel free to change the options as you see fit.

## Testing Fail2Ban

Now just try to login to bitwarden using any email (it doesnt have to be a valid email, just an email format)
If it works correctly and your IP is banned, you can unban the ip by running:
```
sudo fail2ban-client set bitwarden unbanip XX.XX.XX.XX
```
If Fail2Ban does not appear to be functioning, verify that the path to the Bitwarden log file is correct. For Docker: If the specified log file is not being generated and/or updated, make sure the `EXTENDED_LOGGING` env variable is set to true (which is default) and that the path to the log file is the path inside the docker (when you use /bw-data/:/data/ the log file should be in /data/... to be outside the container).

Also verify that the timezone of the docker container matches the timezone of the host. Check this by comparing the time shown in the logfile with the host OS time. If they differ, there are various ways to fix this.  One option is to start docker with the option ```-e "TZ=<timezone>"```. A list of valid timezones is here under the column heading'timezone database name': [https://en.wikipedia.org/wiki/List_of_tz_database_time_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) (ie -e "TZ=Australia/Melbourne")

If you are using podman instead of docker it seems that setting the timezone via ```-e "TZ=<timezone>"``` does not work. This can be solved (when using the alpine image) by following this guide: [https://wiki.alpinelinux.org/wiki/Setting_the_timezone](https://wiki.alpinelinux.org/wiki/Setting_the_timezone).

## Setting Up Fail2Ban for the Admin Page

If you've enabled the admin console by setting the `ADMIN_TOKEN` environment variable, you can prevent an attacker brute-forcing the admin token using Fail2Ban. Following the same process as for the web vault, create the following filter in `/etc/fail2ban/filter.d/bitwarden-admin.conf`:

```
[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Invalid admin token\. IP: <ADDR>.*$
ignoreregex =
```

Then create the following jail configuration in `/etc/fail2ban/jail.d/bitwarden-admin.local` (note that this example uses the `action` directive for the Docker image--modify it if you're using the binary build):

```
[bitwarden-admin]
enabled = true
port = 80,443
filter = bitwarden-admin
action = iptables-allports[name=bitwarden, chain=FORWARD]
logpath = /path/to/bitwarden.log
maxretry = 3
bantime = 14400
findtime = 14400
```

## SELinux Problems
When you are using SELinux it is possible that SELinux hinders fail2ban to read the logs. If so, follow these steps:
`sudo tail /var/log/audit/audit.log`. There you should see something along the lines of this (of course the actual audit ID will vary in your case): 
```
type=AVC msg=audit(1571777936.719:2193): avc:  denied  { search } for  pid=5853 comm="fail2ban-server" name="containers" dev="dm-0" ino=1144588 scontext=system_u:system_r:fail2ban_t:s0 tcontext=unconfined_u:object_r:container_var_lib_t:s0 tclass=dir permissive=0
```   
To actually find out the reason you can use `grep 'type=AVC msg=audit(1571777936.719:2193)' /var/log/audit/audit.log | audit2why`. `audit2allow -a` will give you specific instructions on how to create a module and allow fail2ban to access these logs. Follow these steps and you're done! fail2ban should now work correctly.

## Setup on Synology
Synology, due to DSM system need a bit more work. The main constrains are:

1. The embeded IP ban system does not work on Docker's containers
2. The iptables embeded do no support the `REJECT` instruction
3. The Docker GUI does not allow some advanced settings

I choosed to rely on [crazy-max/docker-fail2ban](https://github.com/crazy-max/docker-fail2ban). Please adapt the following to your context

`mkdir /volumeX/docker/fail2ban`  
`touch /volumeX/docker/fail2ban/action.d/iptables-common.local`  
Copy and paste the following content - this replace `REJECT` by `DROP`  
````
[Init]
blocktype = DROP
[Init?family=inet6]
blocktype = DROP
````
`touch /volumeX/docker/fail2ban/filter.d/bitwarden.conf`  
Copy and paste the following content
````
[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$
ignoreregex =
````
`touch /volumeX/docker/fail2ban/jail.d/bitwarden.conf`  
Copy and paste the following content
````
[DEFAULT]
ignoreip = 127.0.0.1/8 192.168.0.0/22
bantime = 6400
findtime = 86400
maxretry = 4
backend = auto
action = iptables-allports[name=bitwarden]

[bitwarden]
enabled = true
port = 80,81,443,8081
filter = bitwarden
logpath = /bitwarden/bitwarden.log
````
`touch /volumeX/docker/fail2ban/docker-compose.yml`  
Copy and paste the following content
````
version: '3'
services:
  fail2ban:
    container_name: fail2ban
    restart: always
    image: crazymax/fail2ban:latest
    environment: 
    - TZ=Europe/Paris
    - F2B_DB_PURGE_AGE=30d
    - F2B_LOG_TARGET=/data/fail2ban.log
    - F2B_LOG_LEVEL=DEBUG
    - F2B_IPTABLES_CHAIN=INPUT

    volumes:
    - /volumeX/docker/fail2ban:/data
    - /volumeX/docker/bw-data:/bitwarden:ro

    network_mode: "host"

    privileged: true
    cap_add:
        - NET_ADMIN
        - NET_RAW
````
Run the container using `docker-compose up -d`  
You now have to test the jail

