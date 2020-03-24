Setup Fail2ban will prevent attackers to brute force your vault logins. This is particularly important if your instance is publicaly available.

## Pre-requisite

- Commands are using `vi`. The basics can be found [there](https://pc.net/resources/commands/vi). However, you can use whatever text editor you want.
- From Release 1.5.0, Bitwarden_rs supports logging to file. Please set this up : [[Logging|logging]]
- Try to log to web vault with a false account and check the log files for folowing format
````
	[YYYY-MM-DD hh:mm:ss][bitwarden_rs::api::identity][ERROR] Username or password is incorrect. Try again. IP: XXX.XXX.XXX.XXX. Username: email@domain.com.
````

## Installation
### Debian / Ubuntu / Raspian
```
	sudo apt-get install fail2ban -y
```
### Fedora / Centos
EPEL repository is necessary (CentOS 7)  
```
	sudo yum install epel-release
	sudo yum install fail2ban -y
```
### Synology DSM
With Synology, a bit more work is needed for various reasons. The main issues are:

1. The embeded IP ban system does not work for Docker's containers
2. The iptables embeded do no support the `REJECT` blocktype
3. The Docker GUI does not allow some advanced settings
4. Modifying system configuration is not upgrade-proof

Therefore, we will use Fail2ban in a docker container. [crazy-max/docker-fail2ban](https://github.com/crazy-max/docker-fail2ban) provides a good solution and the Synology's docker GUI will be ignored. From command line through SSH, here the steps. As convention `volumeX` is to be adapted to your Synology's config.  

0. Get root
````
	sudo -i
````

1. Creating persistent folders

````
	mkdir -p /volumeX/docker/fail2ban/action.d/
	mkdir -p /volumeX/docker/fail2ban/jail.d/
	mkdir -p /volumeX/docker/fail2ban/filter.d/
````

2. Replace `REJECT` by `DROP` blocktype
````
	vi /volumeX/docker/fail2ban/action.d/iptables-common.local
	
	Copy and paste the following content  

	[Init]
	blocktype = DROP
	[Init?family=inet6]
	blocktype = DROP
````
3. Create docker-compose file
````
	vi /volumeX/docker/fail2ban/docker-compose.yml
	
	Copy and paste the following content

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
		- /volumeX/docker/bw-data:/bitwarden:ro

		network_mode: "host"

		privileged: true
		cap_add:
			- NET_ADMIN
			- NET_RAW
````
4. Start the container using command line
````
	cd /volumeX/docker/fail2ban
	docker-compose up -d
````
You should see the container running in Synolog's Docker GUI. You will have to reload after configuring the filters and jails

## Setup for web vault

As a convention, `path_f2b` means the path needed for Fail2ban to work. This depends on your system. E.g. on Synology, we are atlking about `/volumeX/docker/fail2ban/` where on some other systems we are talking about `/etc/fail2ban/`

### Filter
Create and fill the following file
````
	vi path_f2b/filter.d/bitwarden.local
	
	Copy and paste the following content
	
	[INCLUDES]
	before = common.conf

	[Definition]
	failregex = ^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$
	ignoreregex =
````

If you get the following error message `in fail2ban.log` (CentOS 7, Fail2Ban v0.9.7)

If you get the following error message in fail2ban.log  
`fail2ban.filter         [5291]: ERROR   No 'host' group in '^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$'`  
Please Use `<HOST>` instead of `<ADDR>` in ``bitwarden.local`

### Jail
Create and fill the following file
````
	vi path_f2b/jail.d/bitwarden.local
	
	Copy and paste the following content
	
	[bitwarden]
	enabled = true
	port = 80,443,8081
	filter = bitwarden
	action = iptables-allports[name=bitwarden]
	logpath = /path/to/bitwarden/log
	maxretry = 3
	bantime = 14400
	findtime = 14400
````
Note: Docker uses the FORWARD chain instead of the default INPUT chain. Therefore use the following action when using Docker:
```
action = iptables-allports[name=bitwarden, chain=FORWARD]
```
**NOTE**:  
Do not use this if you use a reverse proxy before docker container. If proxy, like apache2 or nginx is used, use the ports of the proxy and do not use chain=FORWARD, only when using Docker **without** proxy!

**NOTE on the NOTE above**:  
Thats at least not true for running on Docker (CentOS 7) with caddy as reverse proxy. chain=FORWARD is absolutely fine and working with caddy as reverse proxy.

Feel free to change the options as you see fit.


## Setup for admin page
If you've enabled the admin console by setting the `ADMIN_TOKEN` environment variable, you can prevent an attacker brute-forcing the admin token using Fail2Ban. The process is the same as for the web vault.

### Filter
Create and fill the following file
````
	vi path_f2b/filter.d/bitwarden-admin.local
	
	Copy and paste the following content
	
	[INCLUDES]
	before = common.conf

	[Definition]
	failregex = ^.*Invalid admin token\. IP: <ADDR>.*$
	ignoreregex =
````
### Jail
Create and fill the following file
````
	vi path_f2b/jail.d/bitwarden-admin.local
	
	Copy and paste the following content
	
	[bitwarden-admin]
	enabled = true
	port = 80,443
	filter = bitwarden-admin
	action = iptables-allports[name=bitwarden]
	logpath = /path/to/bitwarden.log
	maxretry = 3
	bantime = 14400
	findt
````
Note: Docker uses the FORWARD chain instead of the default INPUT chain. Therefore use the following action when using Docker:
```
action = iptables-allports[name=bitwarden, chain=FORWARD]
```

## Testing Fail2Ban

Now just try to login to bitwarden using any email (it doesnt have to be a valid email, just an email format)
If it works correctly and your IP is banned, you can unban the ip by running:
```
sudo fail2ban-client set bitwarden unbanip XX.XX.XX.XX
```
If Fail2Ban does not appear to be functioning, verify that the path to the Bitwarden log file is correct. For Docker: If the specified log file is not being generated and/or updated, make sure the `EXTENDED_LOGGING` env variable is set to true (which is default) and that the path to the log file is the path inside the docker (when you use /bw-data/:/data/ the log file should be in /data/... to be outside the container).

Also verify that the timezone of the docker container matches the timezone of the host. Check this by comparing the time shown in the logfile with the host OS time. If they differ, there are various ways to fix this.  One option is to start docker with the option ```-e "TZ=<timezone>"```. A list of valid timezones is here under the column heading'timezone database name': [https://en.wikipedia.org/wiki/List_of_tz_database_time_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) (ie -e "TZ=Australia/Melbourne")

If you are using podman instead of docker it seems that setting the timezone via ```-e "TZ=<timezone>"``` does not work. This can be solved (when using the alpine image) by following this guide: [https://wiki.alpinelinux.org/wiki/Setting_the_timezone](https://wiki.alpinelinux.org/wiki/Setting_the_timezone).


## SELinux Problems
When you are using SELinux it is possible that SELinux hinders fail2ban to read the logs. If so, follow these steps:
`sudo tail /var/log/audit/audit.log`. There you should see something along the lines of this (of course the actual audit ID will vary in your case): 
```
type=AVC msg=audit(1571777936.719:2193): avc:  denied  { search } for  pid=5853 comm="fail2ban-server" name="containers" dev="dm-0" ino=1144588 scontext=system_u:system_r:fail2ban_t:s0 tcontext=unconfined_u:object_r:container_var_lib_t:s0 tclass=dir permissive=0
```   
To actually find out the reason you can use `grep 'type=AVC msg=audit(1571777936.719:2193)' /var/log/audit/audit.log | audit2why`. `audit2allow -a` will give you specific instructions on how to create a module and allow fail2ban to access these logs. Follow these steps and you're done! fail2ban should now work correctly.

