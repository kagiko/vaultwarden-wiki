Note that the `docker run` command has a slightly misleading name, as it rather creates a container, rather than just starting it, leading to conflicts when using `docker run` after just stopping the container without removing it. For a plain start, see below.

# Creating the Container

The persistent data is stored under /data inside the container, so the only requirement for persistent deployment using Docker is to mount persistent volume at the path:

```sh
# using Docker:
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 80:80 vaultwarden/server:latest
# using Podman as non-root:
podman run -d --name vaultwarden -v /vw-data/:/data/:Z -e ROCKET_PORT=8080 -p 8080:8080 vaultwarden/server:latest
# using Podman as root:
sudo podman run -d --name vaultwarden -v vw-data:/data/:Z -p 80:80 vaultwarden/server:latest
```


This will preserve any persistent data under `/vw-data/`, you can adapt the path to whatever suits you.

The service will be exposed on host-port 80 or 8080.

For non-x86 hardware or to run specific version, you can [[choose some other image|Which-container-image-to-use]]. 

If your docker/vaultwarden runs on a device with a fixed IP, you can bind the host-port to that specific IP and hence prevent exposing the host-port to the whole world or network. Add the IP address (e.g. 192.168.0.2) in front of the host-port and container-port as follows:

```
# using Docker:
docker run -d --name vaultwarden -v /vw-data/:/data/ -p 192.168.0.2:80:80 vaultwarden/server:latest
```

# Starting the container

If the container has been stopped by `docker stop vaultwarden`, a reboot or any other reason you can just start it up again by using
```
docker start vaultwarden
```

# Customizing container startup

If you have custom startup script(s) you want to run when the container starts, you can mount a single script into the container as `/etc/vaultwarden.sh` and/or a directory of scripts as `/etc/vaultwarden.d`. In the latter case, only files with an `.sh` extension are run, so files with other extensions (e.g., data/config files) can reside in the same dir. (See [start.sh](https://github.com/dani-garcia/vaultwarden/blob/master/docker/start.sh) for details on exactly how it works.)

A custom startup script can be useful for patching web vault files or installing additional packages, CA certificates, etc. without having to build and maintain your own Docker image.

## Example

Suppose your script is named `init.sh` and contains the following:
```
echo "starting up"
```

You can run the script on startup like this:
```
docker run -d --name vaultwarden -v $(pwd)/init.sh:/etc/vaultwarden.sh <other docker args...> vaultwarden/server:latest
```

If you run `docker logs vaultwarden`, you should now see `starting up` as the first line of the output.

Note that the init scripts are run each time the container starts (not just the first time), so these scripts should generally be idempotent (i.e., you can run the scripts more than once without undesirable/erroneous behavior). If your scripts don't naturally have this property, you can do something like this:
```
if [ ! -e /.init ]; then
  touch /.init

  # run your init steps...
fi
```
