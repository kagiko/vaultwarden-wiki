Clone the repository, then from the root of the repository run to build with default sqlite backend:

```sh
# Build the docker image:
docker build -t vaultwarden .
```

To build with MySQL backend run:
```sh
# Build the docker image:
docker build -t vaultwarden --build-arg DB=mysql .
``` 

To build with Postgresql backend run:
```sh
# Build the docker image:
docker build -t vaultwarden --build-arg DB=postgresql .
``` 
in docker-compose.yml it looks like
```...
  bitwarden:
    # image: vaultwarden/server-postgresql:latest
    image: vaultwarden
    build: 
      context: vaultwarden
      args: 
        DB: postgresql
```
