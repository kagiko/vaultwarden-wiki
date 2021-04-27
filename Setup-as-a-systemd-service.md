These instructions require you to have [[compiled the vaultwarden binary|Building-binary]]. If you generated a docker image, you may want to look at [[Running with systemd-docker | Running-with-systemd-docker]]
## Setup
Making vaultwarden start on system startup and use the other facilities of systemd (e.g. isolation, logging,...) requires a `.service` file. The following is a usable starting point:
```ini
[Unit]
Description=Bitwarden Server (Rust Edition)
Documentation=https://github.com/dani-garcia/vaultwarden
# If you use a database like mariadb,mysql or postgresql, 
# you have to add them like the following and uncomment them 
# by removing the `# ` before it. This makes sure that your 
# database server is started before vaultwarden ("After") and has 
# started successfully before starting vaultwarden ("Requires").

# Only sqlite
After=network.target

# MariaDB
# After=network.target mariadb.service
# Requires=mariadb.service

# Mysql
# After=network.target mysqld.service
# Requires=mysqld.service

# PostgreSQL
# After=network.target postgresql.service
# Requires=postgresql.service


[Service]
# The user/group vaultwarden is run under. the working directory (see below) should allow write and read access to this user/group
User=vaultwarden
Group=vaultwarden
# The location of the .env file for configuration
EnvironmentFile=/etc/vaultwarden.env
# The location of the compiled binary
ExecStart=/usr/bin/vaultwarden
# Set reasonable connection and process limits
LimitNOFILE=1048576
LimitNPROC=64
# Isolate vaultwarden from the rest of the system
PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
# Only allow writes to the following directory and set it to the working directory (user and password data are stored here)
WorkingDirectory=/var/lib/vaultwarden
ReadWriteDirectories=/var/lib/vaultwarden
# Allow vaultwarden to bind ports in the range of 0-1024
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```
Change all paths to match your installation (`WorkingDirectory` and `ReadWriteDirectory` should be the same),
name this file `vaultwarden.service` and put it into `/etc/systemd/system`. 

If you have to change an existing systemd file (which was provided to you by the package you installed), you can add your changes by using 
```
$ sudo systemctl edit vaultwarden.service
```
To make systemd aware of your new file or any changes you made, run
```
$ sudo systemctl daemon-reload
```
## Usage
To start this "service", run
```
$ sudo systemctl start vaultwarden.service
```

To enable autostart, run
```
$ sudo systemctl enable vaultwarden.service
```
In the same way you can `stop`, `restart` and `disable` the service.
### Updating vaultwarden
After compiling the new version of vaultwarden, you can copy the compiled (new) binary and replace the existing (old) binary and then restart the service:
```
$ sudo systemctl restart vaultwarden.service
```
### Uninstalling vaultwarden
Before doing anything else, you should stop and disable the service:
```
$ sudo systemctl disable --now vaultwarden.service
```
Then you can delete the binary, the `.env` file, the web-vault folder (if installed) and the user data (if necessary). Remember to also remove specially created users,groups and firewall rules (if needed) and the systemd file.

After removing the systemd file you should make systemd aware of it via:
```
$ sudo systemctl daemon-reload
```

### Logging and status view
If you want to see the logging output, run
```
$ journalctl -u vaultwarden.service
```
or to see a more concise state of the service, run
```
$ systemctl status vaultwarden.service
```

## Troubleshooting
### Sandboxing options with older systemd versions
In RHEL 7 (and debian 8), the used systemd does not support some of the used isolation options. ([#445](https://github.com/dani-garcia/vaultwarden/issues/445),[#363](https://github.com/dani-garcia/vaultwarden/issues/363))
This can result in one of the following errors:
```
Failed at step NAMESPACE spawning /home/vaultwarden/vaultwarden: Permission denied
```
or 
```
Failed to parse protect system value
```
To work around this you can comment out some or all of these settings by putting a `#` in front of the lines containing
`PrivateTmp`, `PrivateDevices`, `ProtectHome`, `ProtectSystem` and `ReadWriteDirectories`. While commenting out all of them will probably work, it's not recommended as these are security measures which are good to have. To see which options your systemd supports, look at the output of
```
$ systemctl --version
```
to determine your systemd version and compare with [systemd/NEWS.md](https://github.com/systemd/systemd/blob/master/NEWS).

After editing your `.service` file, don't forget to 
```
$ sudo systemctl daemon-reload
```
before (re-)starting your service.

### Service fails to start

The following error shows in the systemd journal (`journalctl -eu vaultwarden.service`):

```
Feb 18 05:29:10 staging-bitwarden systemd[1]: Started Bitwarden Server (Rust Edition).
Feb 18 05:29:10 staging-bitwarden systemd[49506]: vaultwarden.service: Failed to execute command: Resource temporarily unavailable
Feb 18 05:29:10 staging-bitwarden systemd[49506]: vaultwarden.service: Failed at step EXEC spawning /usr/bin/vaultwarden: Resource temporarily unavailable
Feb 18 05:29:10 staging-bitwarden systemd[1]: vaultwarden.service: Main process exited, code=exited, status=203/EXEC
Feb 18 05:29:10 staging-bitwarden systemd[1]: vaultwarden.service: Failed with result 'exit-code'.
```

This is known to occur when vaultwarden is running inside a container (LXC, et al) or natively. The parameter `LimitNPROC=64` in the service file prevents the service from starting. Commenting out that particular parameter results in the service starting correctly.

**Note**: A systemd override file will not work, the line must be commented out/removed. The easiest way to do this is via
```
# systemctl edit --full vaultwarden.service
```
then reloading the daemon & restarting.


## More information
For more information on .service files, see the manpages of [systemd.service](https://www.freedesktop.org/software/systemd/man/systemd.service.html) and (for the security configuration) [systemd.exec](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)