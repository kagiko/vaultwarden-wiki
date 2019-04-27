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

To reduce the amount of log messages, you can set the log level to 'critical' (default is 'normal') and disable the extended logging (enabled by default). The [Log level](https://api.rocket.rs/v0.3/rocket/config/enum.LoggingLevel.html) can be adjusted with the environment variable `ROCKET_LOG`. Extended logging can be deactivated by setting the environment variable `EXTENDED_LOGGING` to 'false'. Be advised, disabling `EXTENDED_LOGGING` will stop the log file from being written to.

```sh
docker run -d --name bitwarden \
...
  -e ROCKET_LOG=critical -e EXTENDED_LOGGING=false \
...
```