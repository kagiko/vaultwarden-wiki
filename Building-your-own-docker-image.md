Clone the repository, then from the root of the repository run to build with default sqlite backend:

## Recommended way to build

Please read the documentation provided in the docker directory for the latest information on how to build Vaultwarden locally.<br>
This can be done via docker or podman. See: https://github.com/dani-garcia/vaultwarden/tree/main/docker


## Simple ways to build

```sh
# Build the docker image with all databases supported:
docker buildx build -t vaultwarden .
```

To build with SQLite backend only run:
```bash
# Build the docker image:
docker buildx build -t vaultwarden --build-arg DB=sqlite .
```


To build with MySQL backend only run:
```bash
# Build the docker image:
docker buildx build -t vaultwarden --build-arg DB=mysql .
```

To build with Postgresql backend only run:
```bash
# Build the docker image:
docker buildx build -t vaultwarden --build-arg DB=postgresql .
```

in docker-compose.yml it looks like
```yaml
  vaultwarden:
    image: vaultwarden
    build:
      context: vaultwarden
      args:
        DB: postgresql
```
