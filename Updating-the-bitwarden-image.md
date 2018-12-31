Updating is straightforward, you just make sure to preserve the mounted volume. If you used the bind-mounted path as in the example above, you just need to `pull` the latest image, `stop` and `rm` the current container and then start a new one the same way as before:

```sh
# Pull the latest version
docker pull mprasil/bitwarden:latest

# Stop and remove the old container
docker stop bitwarden
docker rm bitwarden

# Start new container with the data mounted
docker run -d --name bitwarden -v /bw-data/:/data/ -p 80:80 mprasil/bitwarden:latest
```
Then visit [http://localhost:80](http://localhost:80)

In case you didn't bind mount the volume for persistent data, you need an intermediate step where you preserve the data with an intermediate container:

```sh
# Pull the latest version
docker pull mprasil/bitwarden:latest

# Create intermediate container to preserve data
docker run --volumes-from bitwarden --name bitwarden_data busybox true

# Stop and remove the old container
docker stop bitwarden
docker rm bitwarden

# Start new container with the data mounted
docker run -d --volumes-from bitwarden_data --name bitwarden -p 80:80 mprasil/bitwarden:latest

# Optionally remove the intermediate container
docker rm bitwarden_data

# Alternatively you can keep data container around for future updates in which case you can skip last step.
```