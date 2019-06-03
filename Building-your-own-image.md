Clone the repository, then from the root of the repository run to build with default sqlite backend:

```sh
# Build the docker image:
docker build -t bitwarden_rs .
```

To build with MySQL backend run:
```sh
# Build the docker image:
docker build -t bitwarden_rs --build-arg DB=sqlite .
``` 