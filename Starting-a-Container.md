The persistent data is stored under /data inside the container, so the only requirement for persistent deployment using Docker is to mount persistent volume at the path:

```sh
# using Docker:
docker run -d --name bitwarden -v /bw-data/:/data/ -p 80:80 bitwardenrs/server:latest
# using Podman as non-root:
podman run -d --name bitwarden -v /bw-data/:/data/:Z -p 8080:8080 bitwardenrs/server:latest
# using Podman as root:
sudo podman run -d --name bitwarden -v /bw-data/:/data/:Z -p 80:8080 bitwardenrs/server:latest
```


This will preserve any persistent data under `/bw-data/`, you can adapt the path to whatever suits you.

The service will be exposed on host-port 80 or 8080.

For non-x86 hardware or to run specific version, you can [[choose some other image|Which-Docker-image-to-use]]. 

If your docker/bitwarden_rs runs on a device with a fixed IP, you can bind the host-port to that specific IP and hence prevent exposing the host-port to the whole world or network. Add the IP address (e.g. 192.168.0.2) in front of the host-port and container-port as follows:

```
# using Docker:
docker run -d --name bitwarden -v /bw-data/:/data/ -p 192.168.0.2:80:80 bitwardenrs/server:latest
```