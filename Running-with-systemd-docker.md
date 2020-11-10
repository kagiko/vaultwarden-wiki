These instructions allow you to have systemd manage the lifecycle of the docker container, if you prefer.

First, install the `systemd-docker` package using your system package manager.
This is a wrapper which improves docker integration with systemd.

For full instructions and configuration options, see the [GitHub repository](https://github.com/ibuildthecloud/systemd-docker).

As root, create `/etc/systemd/system/bitwarden.service` using your preferred editor with the following contents:

```ini
[Unit]
Description=Bitwarden
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker pull bitwardenrs/server:latest
ExecStart=/usr/bin/docker run -d \
  -p 8080:80 \
  -p 8081:3012 \
  --env-file /opt/.bitwarden.env \
  -v /opt/bw-data:/data/ \
  --restart=on-failure --name bitwarden bitwardenrs/server:latest
ExecStop=/usr/bin/docker stop bitwarden
ExecStopPost=-/usr/bin/docker rm bitwarden
Restart=Always
RestartSec=30s
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

Adjust the above example as necessary. In particular, pay attention to the `-p` and `-v` options,
as these control the port and volume bindings between the container and the host.
Also make sure to provide a `--env-file` with your configurations, or type out all your configurations via `-e KEY=VALUE` directly.

Explanation of options which may not be self-explanatory:

- A `TimeoutStartSec` value of 0 stops systemd from considering the service failed
  after waiting for the default startup time. This is required as it may take a while for the `docker pull` in `ExecStartPre` to finish.
- `ExecStartPre`: Pull the docker tag before running.
- `ExecStopPost`: Delete the container (to make sure we can start again next time). The reason we do that is because systemd is monitoring the docker service instead of the individual container. As such we tell the docker service to restart the container `unless-stopped`. That is basically like `--restart=Always`, but excluding when the docker service stopped (or the container was halted). This allows us to only restart the service `Restart=Always` with systemd when the docker service stopped.
- A `Type` value of `notify` tells systemd to expect a notification from the service that it is ready.
- A `NotifyAccess` value of `all` is required by `systemd-docker`.

## Setting environment variables

It's possible to directly specify environment variables in the unit file in two ways:

- Using an `Environment` directive in the `[Service]` block.
- Using the `-e` option of `docker`. In this case, you can omit the `--env` option shown in the example above.

To verify that your environment variables are set correctly, check the output of `systemctl show bitwarden.service`
for an `Environment` line.

It's also possible to store environment variables in a separate file using the `EnvironmentFile` directive in the unit file. In this case remember to set the `--env` option in the docker commandline as shown above, otherwise the environment file will not be processed.

Systemd can source a file of the form:

```shell
Key="Value"
```
You can find more environment settings and the correct syntax in this [example env template](https://github.com/dani-garcia/bitwarden_rs/blob/21325b7523a68ab3ae8d435ab5b73176db6155ff/.env.template).

However, the systemd project does not mandate where this file should be stored. Consult your distribution's documentation for the
best location for this file. For example, RedHat based distributions typically place these files in `/etc/sysconfig/`

If you're unsure, just create a file as root in `/etc/` e.g. `/etc/bitwarden.service.conf`.

In your unit file, add an `EnvironmentFile` directive in the `[Service]` block, the value being the full path to the
file created above. Example:

```ini
[Unit]
Description=Bitwarden
After=docker.service
Requires=docker.service

[Service]
EnvironmentFile=/etc/bitwarden.service.conf
TimeoutStartSec=0
-snip-
```

## Running the service

After the above installation and configuration is complete, reload systemd using `sudo systemctl daemon-reload`.
Then, start the Bitwarden service using `sudo systemctl start bitwarden`.

To have the service start with the system, use `sudo systemctl enable bitwarden`.

Verify that the container has started using `systemctl status bitwarden`.

If you are getting the `json: cannot unmarshal object into Go value of type string` error when starting the service, you should compile the systemd-docker binary yourself using a recent version of Go, see [this issue](https://github.com/ibuildthecloud/systemd-docker/issues/50).