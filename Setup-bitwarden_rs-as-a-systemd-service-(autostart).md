Making bitwarden_rs start on system startup and use the other facilities of systemd (e.g. isolation, logging,...) requires a `.service` file. The following is a usable starting point:
```
[Unit]
Description=Bitwarden Server (Rust Edition)
Documentation=https://github.com/dani-garcia/bitwarden_rs
After=network.target

[Service]
# The user/group bitwarden_rs is run under. the working directory (see below) should allow write and read access to this user/group
User=bitwarden_rs
Group=bitwarden_rs
# The location of the .env file for configuration
EnvironmentFile=/etc/bitwarden_rs.env
# The location of the compiled binary
ExecStart=/usr/bin/bitwarden_rs
# Set reasonable connection and process limits
LimitNOFILE=1048576
LimitNPROC=64
# Isolate bitwarden_rs from the rest of the system
PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
# Only allow writes to the following directory and set it to the working directory (user and password data are stored here)
WorkingDirectory=/var/lib/bitwarden_rs
ReadWriteDirectories=/var/lib/bitwarden_rs
# Allow bitwarden_rs to bind ports in the range of 0-1024
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```
Change all paths to match your installation (`WorkingDirectory` and `ReadWriteDirectory` should be the same),
name this file `bitwarden_rs.service` and put it into `/etc/systemd/system` .
To make systemd aware of it, run
```
$ sudo systemctl daemon-reload
```

To start this new "service", run
```
$ sudo systemctl start bitwarden_rs.service
```

To enable autostart, run
```
$ sudo systemctl enable bitwarden_rs.service
```
In the same way you can `stop`, `restart` and `disable` the service.
If you want to see the logging output, run
```
$ journalctl -u bitwarden_rs.service
```
or to see a more concise state of the service, run
```
$ systemctl status bitwarden_rs.service
```

For more information on .service files, see the manpages of [systemd.service](https://www.freedesktop.org/software/systemd/man/systemd.service.html) and (for the security configuration) [systemd.exec](https://www.freedesktop.org/software/systemd/man/systemd.exec.html)