The persistent data is stored under /data inside the container, so the only requirement for persistent deployment using Docker is to mount persistent volume at the path:

```
docker run -d --name bitwarden -v /bw-data/:/data/ -p 80:80 bitwardenrs/server:latest
```

This will preserve any persistent data under `/bw-data/`, you can adapt the path to whatever suits you.

The service will be exposed on port 80.

For non-x86 hardware or to run specific version, you can [[choose some other image|Which-Docker-image-to-use]]. 