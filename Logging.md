Logging to a file is supported as of 1.5.0. You can specify the path to the log file with the `LOG_FILE` environment variable:

```sh
docker run -d --name bitwarden \
...
  -e LOG_FILE=/data/bitwarden.log \
...
```

Note that if you're using the docker image, you'll most likely want to use a file path that is mounted from the host OS (such as the data folder).