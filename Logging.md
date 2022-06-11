vaultwarden logs only to [standard output](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout)) (stdout) by default. You can also configure it to log to a file or Syslog.

## Logging to a file

Logging to a file is supported as of version 1.5.0. You can specify the path to the log file with the `LOG_FILE` environment variable:

```sh
docker run -d --name vaultwarden \
...
  -e LOG_FILE=/data/vaultwarden.log \
...
```

When this environment variable is set, log messages will be logged to both stdout and the log file. If you're running in Docker, you'll most likely want to use a file path that is mounted from the Docker host (such as the `data` folder); otherwise, your log file will be lost (or at least hard to find) if the container is restarted or removed.

## Logging to Syslog

You can use Syslog with the `USE_SYSLOG` environment variable while alse setting `EXTENDED_LOGGING=true`:

```sh
docker run -d --name vaultwarden \
...
  -e USE_SYSLOG=true -e EXTENDED_LOGGING=true \
...
```

When this environment variable is set, log messages will be logged to both stdout and Syslog.

## Changing the log level

To reduce the amount of log messages, you can set the log level to 'warn' (default is 'info'). The [Log level](https://docs.rs/log/0.4.7/log/enum.Level.html#variants) can be adjusted with the environment variable `LOG_LEVEL` while also setting `EXTENDED_LOGGING=true`. NOTE: Using the log level "warn" or "error" still allows [Fail2Ban](https://github.com/dani-garcia/vaultwarden/wiki/Fail2Ban-Setup) to work properly.

`LOG_LEVEL` options are: "trace", "debug", "info", "warn", "error" or "off".

```sh
docker run -d --name vaultwarden \
...
  -e LOG_LEVEL=warn -e EXTENDED_LOGGING=true \
...
```

## Viewing logs

If running in Docker: `docker logs <container-name>`

If running via `systemd`: `journalctl -u vaultwarden.service` (or whatever your service is named)

Otherwise, check where standard output is being redirected, or set the `LOG_FILE` environment variable and view that file.