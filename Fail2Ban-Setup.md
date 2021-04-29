Setup Fail2ban will prevent attackers to brute force your vault logins. This is particularly important if your instance is publicaly available.

## Table of Contents
- [Pre-requisite](#pre-requisite)
- [Installation](#installation)
	* [Debian / Ubuntu / Raspberry Pi OS](#debian--ubuntu--raspberry-pi-os)
	* [Fedora / Centos](#fedora--centos)
	* [Synology DSM](#synology-dsm)
- [Setup for web vault](#setup-for-web-vault)
	* [Filter](#filter)
	* [Jail](#jail)
- [Setup for admin page](#setup-for-admin-page)
	* [Filter](#filter-1)
	* [Jail](#jail-1)
- [Testing Fail2Ban](#testing-fail2ban)
- [SELinux Problems](#selinux-problems)

## Pre-requisite
- Filenames are at the top of each code block.
- From Release 1.5.0, Vaultwarden supports logging to file. Please set this up : [[Logging|logging]]
- Try to log to web vault with a false account and check the log files for following format
```
[YYYY-MM-DD hh:mm:ss][vaultwarden::api::identity][ERROR] Username or password is incorrect. Try again. IP: XXX.XXX.XXX.XXX. Username: email@domain.com.
```

## Installation
### Debian / Ubuntu / Raspberry Pi OS
```bash
sudo apt-get install fail2ban -y
```

### Fedora / Centos
EPEL repository is necessary (CentOS 7)  
```bash
sudo yum install epel-release
sudo yum install fail2ban -y
```

### Synology DSM
With Synology, a bit more work is needed for various reasons. The full solution is pushed with Docker Compose [there](https://github.com/sosandroid/docker-fail2ban-synology). The main issues are:

1. The embedded IP ban system does not work for Docker's containers
2. The iptables embedded do no support the `REJECT` blocktype
3. The Docker GUI does not allow some advanced settings
4. Modifying system configuration is not upgrade-proof

Therefore, we will use Fail2ban in a Docker container. [Crazy-max/docker-fail2ban](https://github.com/crazy-max/docker-fail2ban) provides a good solution and the Synology's Docker GUI will be ignored. From command line through SSH, here the steps. As a convention `volumeX` is to be adapted to your Synology's config.  

0. Get root
```bash
sudo -i
```

1. Creating persistent folders
```bash
mkdir -p /volumeX/docker/fail2ban/action.d/
mkdir -p /volumeX/docker/fail2ban/jail.d/
mkdir -p /volumeX/docker/fail2ban/filter.d/
```

2. Replace `REJECT` by `DROP` blocktype
```INI
# /volumeX/docker/fail2ban/action.d/iptables-common.local

[Init]
blocktype = DROP
[Init?family=inet6]
blocktype = DROP
```

3. Create `docker-compose` file
```yml
# /volumeX/docker/fail2ban/docker-compose.yml

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
      - F2B_LOG_LEVEL=INFO
      - F2B_IPTABLES_CHAIN=INPUT

    volumes:
      - /volumeX/docker/fail2ban:/data
      - /volumeX/docker/vw-data:/bitwarden:ro

    network_mode: "host"

    privileged: true
    cap_add:
      - NET_ADMIN
      - NET_RAW
```

4. Start the container using command line
```bash
cd /volumeX/docker/fail2ban
docker-compose up -d
```

You should see the container running in Synology's Docker GUI. You will have to reload after configuring the filters and jails

## Setup for web vault
As a convention, `path_f2b` means the path needed for Fail2ban to work. This depends on your system. E.g. on Synology, we are talking about `/volumeX/docker/fail2ban/` where on some other systems we are talking about `/etc/fail2ban/`

### Filter

Create and fill the following file

```INI
# path_f2b/filter.d/vaultwarden.local

[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$
ignoreregex =
```

**Tip:** If you get the following error message in `fail2ban.log` (CentOS 7, Fail2Ban v0.9.7)  
`fail2ban.filter         [5291]: ERROR   No 'host' group in '^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$'`  
Please Use `<HOST>` instead of `<ADDR>` in `vaultwarden.local`

**Tip:** If you see 127.0.0.1 as the IP address of failed logins in bitwarden.log, then you're probably using a reverse proxy and fail2ban won't work correctly:
```
[YYYY-MM-DD hh:mm:ss][vaultwarden::api::identity][ERROR] Username or password is incorrect. Try again. IP: 127.0.0.1. Username: email@example.com.
```
To remedy this, forward the true remote address to vaultwarden via the X-Real-IP header. How to do this varies depending on the proxy you use. For example, in Caddy 2.x, when you define the reverse-proxy, define `header_up X-Real-IP {remote_host}`. See [Proxy examples](https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples) for more info.

### Jail

Create and fill the following file
```INI
# path_f2b/jail.d/vaultwarden.local

[vaultwarden]
enabled = true
port = 80,443,8081
filter = vaultwarden
banaction = %(banaction_allports)s
logpath = /path/to/bitwarden.log
maxretry = 3
bantime = 14400
findtime = 14400
```

Note: Docker uses the FORWARD chain instead of the default INPUT chain. Therefore replace the `banaction` line with the following `action` when using Docker:
```INI
action = iptables-allports[name=vaultwarden, chain=FORWARD]
```

**NOTE**:  
Do not use this if you use a reverse proxy before Docker container. If proxy, like apache2 or nginx is used, use the ports of the proxy and do not use `chain=FORWARD`, only when using Docker **without** proxy!

**NOTE on the NOTE above**:  
That's at least not true for running on Docker (CentOS 7) with caddy as reverse proxy. `chain=FORWARD` is absolutely fine and working with caddy as reverse proxy.

Reload fail2ban for changes to take effect:

```bash
sudo systemctl reload fail2ban
```

Feel free to change the options as you see fit.

## Setup for admin page
If you've enabled the admin console by setting the `ADMIN_TOKEN` environment variable, you can prevent an attacker from brute-forcing the admin token using Fail2Ban. The process is the same as for the web vault.

### Filter

Create and fill the following file
```INI
# path_f2b/filter.d/vaultwarden-admin.local

[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Invalid admin token\. IP: <ADDR>.*$
ignoreregex =
```

**Tip:** If you get the following error message in `fail2ban.log`
`ERROR  NOK: ("No 'host' group in '^.*Invalid admin token\\. IP: <ADDR>.*$'")`  
Please Use `<HOST>` instead of `<ADDR>` in `vaultwarden-admin.local`

### Jail

Create and fill the following file
```INI
# path_f2b/jail.d/vaultwarden-admin.local

[vaultwarden-admin]
enabled = true
port = 80,443
filter = vaultwarden-admin
banaction = %(banaction_allports)s
logpath = /path/to/bitwarden.log
maxretry = 3
bantime = 14400
findtime = 14400
```

Note: Docker uses the FORWARD chain instead of the default INPUT chain. Therefore replace the `banaction` line with the following `action` when using Docker:
```INI
action = iptables-allports[name=vaultwarden-admin, chain=FORWARD]
```

Reload fail2ban for changes to take effect:

```bash
sudo systemctl reload fail2ban
```

## Testing Fail2Ban
Now just try to login to bitwarden using any email (it doesn't have to be a valid email, just an email format)
If it works correctly and your IP is banned, you can unban the IP by running:

Without Docker:  
```bash
# With Docker
sudo docker exec -t fail2ban fail2ban-client set vaultwarden unbanip XX.XX.XX.XX
# Without Docker
sudo fail2ban-client set vaultwarden unbanip XX.XX.XX.XX
```

If Fail2Ban does not appear to be functioning, verify that the path to the Bitwarden log file is correct. For Docker: If the specified log file is not being generated and/or updated, make sure the `EXTENDED_LOGGING` env variable is set to true (which is default) and that the path to the log file is the path inside the Docker (when you use `/bw-data/:/data/` the log file should be in `/data/...` to be outside the container).

Also verify that the timezone of the Docker container matches the timezone of the host. Check this by comparing the time shown in the logfile with the host OS time. If they differ, there are various ways to fix this.  One option is to start Docker with the option `-e "TZ=<timezone>"`. A list of valid timezones is [here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) (eg. `-e "TZ=Australia/Melbourne"`)

If you are using podman instead of Docker it seems that setting the timezone via `-e "TZ=<timezone>"` does not work. This can be solved (when using the alpine image) by following this guide: [https://wiki.alpinelinux.org/wiki/Setting_the_timezone](https://wiki.alpinelinux.org/wiki/Setting_the_timezone).

## SELinux Problems
When you are using SELinux it is possible that SELinux hinders fail2ban to read the logs. If so, follow these steps:
`sudo tail /var/log/audit/audit.log`. There you should see something along the lines of this (of course the actual audit ID will vary in your case): 
```
type=AVC msg=audit(1571777936.719:2193): avc:  denied  { search } for  pid=5853 comm="fail2ban-server" name="containers" dev="dm-0" ino=1144588 scontext=system_u:system_r:fail2ban_t:s0 tcontext=unconfined_u:object_r:container_var_lib_t:s0 tclass=dir permissive=0
```   
To actually find out the reason you can use `grep 'type=AVC msg=audit(1571777936.719:2193)' /var/log/audit/audit.log | audit2why`. `audit2allow -a` will give you specific instructions on how to create a module and allow fail2ban to access these logs. Follow these steps and you're done! fail2ban should now work correctly.