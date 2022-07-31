Updating is straightforward, you just make sure to preserve the mounted volume. If you used the bind-mounted path as in the example [[here|Starting-a-Container]], you just need to `pull` the latest image, `stop` and `rm` the current container and then start a new one the same way as before:

```sh
# Pull the latest version
docker pull vaultwarden/server:latest

# Stop and remove the old container
docker stop vaultwarden
docker rm vaultwarden

# Start new container with the data mounted
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 80:80 vaultwarden/server:latest
```
Then visit [http://localhost:80](http://localhost:80)

In case you didn't bind mount the volume for persistent data, you need an intermediate step where you preserve the data with an intermediate container:

```sh
# Pull the latest version
docker pull vaultwarden/server:latest

# Create intermediate container to preserve data
docker run --volumes-from vaultwarden --name vaultwarden_data busybox true

# Stop and remove the old container
docker stop vaultwarden
docker rm vaultwarden

# Start new container with the data mounted
docker run -d --volumes-from vaultwarden_data --name vaultwarden -p 80:80 vaultwarden/server:latest

# Optionally remove the intermediate container
docker rm vaultwarden_data

# Alternatively you can keep data container around for future updates in which case you can skip last step.
```

You can also use a tool like [Watchtower](https://containrrr.dev/watchtower/) to automate the update process. Watchtower can periodically check for an update to the Docker image, pull the updated image, and recreate the container using the updated image.

## Updating when using docker-compose

```sh
docker-compose down
docker-compose pull
docker-compose up -d
```

## Updating when using systemd service (in this case Debian/Raspbian)

```sh
sudo systemctl restart vaultwarden.service
sudo docker system prune -f
#WARNING this could delete stopped or unused containers, etc. not associated with vaultwarden
#be carefull and look which containers you need

docker ps -a
#shows stopped containers

#WARNING! This will remove:
#        - all stopped #containers
#        - all networks not used by at least one container
#        - all dangling images
#        - all dangling build cache
#you can list docker images with
docker images
#there you see all unused images
#
```
The restart command will stop the container, pull the newest images, run the container again.
The prune command will remove the now old container (-f stands for: Do not ask for confirmation).

Put these into cronjob if you want (time can be changed):
```
$ sudo crontab -e
0 2 * * * sudo systemctl restart vaultwarden.service

0 3 * * * sudo /usr/bin/docker system prune -f
```
Use the command

`which docker` 

if `/usr/bin/docker` is not the correct path to docker

## Updating when using DietPi

[https://dietpi.com/](DietPi) is a lightweight Debian-based distribution (image) for all kinds of devices like Raspberry Pi, Odroid, NanoPi and others.

It offers a GUI for installing various programs including Vaultwarden. That spares the user tinkering with installation commands.

To update a previously installed Vautwarden instance which was installed using the DietPi GUI, use the following command on the DietPi command line:
`dietpi-software reinstall 183`

It is recommended to use DietPi version 8.7 or newer because the update process has been optimised and became considerably faster than in previous versions. 

In case you run Vaultwarden with a customised confirguration file, it will be kept in place reserved by the update script.