As of release 1.5.0, bitwarden_rs supports logging to file. See [[Logging|logging]] for information on how to set this up.

## Logging Failed Login Attempts

After specifying the log file location, failed login attempts will appear in the logs in the following format:

```
[YYYY-MM-DD hh:mm:ss][bitwarden_rs::api::identity][ERROR] Username or password is incorrect. Try again. IP: XXX.XXX.XXX.XXX. Username: email@domain.com.
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
failregex = ^.*Username or password is incorrect\. Try again\. IP: <HOST>\. Username:.*$
ignoreregex =
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

Feel free to change the options as you see fit.

## Testing Fail2Ban

Now just try to login to bitwarden using any email (it doesnt have to be a valid email, just an email format)
If it works correctly and your IP is banned, you can unban the ip by running:
```
sudo fail2ban-client set bitwarden unbanip XX.XX.XX.XX
```

## Setting Up Fail2Ban for the Admin Page

If you've enabled the admin console by setting the `ADMIN_TOKEN` environment variable, you can prevent an attacker brute-forcing the admin token using Fail2Ban. Following the same process as for the web vault, create the following filter in `/etc/fail2ban/filter.d/bitwarden-admin.conf`:

```
[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Unauthorized Error: Invalid admin token\. IP: <HOST>.*$
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
maxretry = 5
bantime = 14400
findtime = 14400
```