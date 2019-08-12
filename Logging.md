# Logging to a file

Logging to a file is supported as of 1.5.0. You can specify the path to the log file with the `LOG_FILE` environment variable:

```sh
docker run -d --name bitwarden \
...
  -e LOG_FILE=/data/bitwarden.log \
...
```

Note that if you're using the docker image, you'll most likely want to use a file path that is mounted from the host OS (such as the data folder).

# Change the log level

To reduce the amount of log messages, you can set the log level to 'warn' (default is 'info'). The [Log level](https://docs.rs/log/0.4.7/log/enum.Level.html#variants) can be adjusted with the environment variable `LOG_LEVEL` while also setting `EXTENDED_LOGGING=true`. NOTE: Using the log level "warn" or "error" still allows [Fail2Ban](https://github.com/dani-garcia/bitwarden_rs/wiki/Fail2Ban-Setup) to work properly.

`LOG_LEVEL` options are: "trace", "debug", "info", "warn", "error" or "off".

```sh
docker run -d --name bitwarden \
...
  -e LOG_LEVEL=warn -e EXTENDED_LOGGING=true \
...
```