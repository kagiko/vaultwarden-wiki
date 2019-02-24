By default `mprasil/bitwarden` is using root user to run service inside the container. There are few things you need to set to run the container as non-root user if you wish to do so:

1. Make sure that the directory, you're mounting inside the container will be writable by the user. For example if you decide to run as `nobody`, the directory needs to be writable by user with id 65534. For other ways to specify user inside the container, see the [docker documentation](https://docs.docker.com/engine/reference/run/#user), in our examples here we will use `nobody`.

```bash
# Make the directory on the host, change this to you preferred path
sudo mkdir /bw-data

# Set the owner using user id. 
# Note that the ownership must match user in /etc/passwd *inside* the container, not on your host
sudo chown 65534 /bw-data

# Give the owner full rights to the folder
sudo chmod u+rwx /bw-data
```

2. Start the container with proper parameters. Define the user and make sure to start with port set to `1024` or higher.

```bash
docker run -d \
  --name bitwarden \
  --user nobody \
  -e ROCKET_PORT=1024 \
  -v /bw-data/:/data/ \
  -p 80:1024 \
  mprasil/bitwarden:latest
```

Notice that the port mapping (`-p 80:1024`) reflects the `ROCKET_PORT` setting. 