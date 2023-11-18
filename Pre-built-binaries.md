Vaultwarden doesn't currently provide standalone binaries as a separate download, but you can extract standalone, statically-linked binaries from the official Alpine-based Docker images. Each Docker image also includes a matching web vault build (which is platform-independent).

## Extracting binaries with Docker installed

Assuming you want to extract binaries for the platform you're running on:
```bash
docker pull docker.io/vaultwarden/server:latest-alpine
docker create --name vw docker.io/vaultwarden/server:latest-alpine
docker cp vw:/vaultwarden .
docker cp vw:/web-vault .
docker rm vw
```

If you want binaries for a different platform (for example, you only have Docker installed on your x86-64 machine, but you want to run vaultwarden on a Raspberry Pi), add the `--platform` option to the `docker pull` command:
```bash
docker pull --platform linux/arm/v7 docker.io/vaultwarden/server:latest-alpine
# Run remaining commands as above.
# Note that the `docker create` command may print a message like:
#   WARNING: The requested image's platform (linux/arm/v7) does not match the detected host platform (linux/amd64)
#   and no specific platform was requested
# This is expected, and isn't cause for concern.
```

## Extracting binaries without Docker installed

If you can't or don't want to install Docker, you can use the [docker-image-extract](https://github.com/jjlin/docker-image-extract) script to pull and extract a Docker image. For example, to pull and extract the x86-64 image:
```bash
$ mkdir vw-image
$ cd vw-image
$ wget https://raw.githubusercontent.com/jjlin/docker-image-extract/main/docker-image-extract
$ chmod +x docker-image-extract
$ ./docker-image-extract vaultwarden/server:latest-alpine
Getting multi-arch manifest list...
Platform linux/amd64 resolved to 'sha256:8e344ffce9aef3a18687d21e53e7355a2d924299029c4af0d94acdd25048f292'...
Getting API token...
Getting image manifest for vaultwarden/server:latest-alpine...
Fetching and extracting layer 96526aa774ef0126ad0fe9e9a95764c5fc37f409ab9e97021e7b4775d82bf6fa...
Fetching and extracting layer cd8bc2d8ff321b604bfbaeaf05d5d003d151dff16ba8c9401b2454ce2523d4a8...
Fetching and extracting layer 822e55e7fca73a44ec8200bcae5ff098df0449884bb1489226104e13d861b1d2...
Fetching and extracting layer 639668723db6e831c0c74eea526afd5dc86fb186ca31b70bcb49bcaf687811c6...
Fetching and extracting layer 12f58a7c9a5158fe8b56a294dd7f597aa564c881d8af327ae84e4fe42f301c81...
Fetching and extracting layer 205fba48648467a1b138596797b694ae18b3506ffd5f77ccb968090fef127ed6...
Image contents extracted into ./output.

$ ls -ld output/{vaultwarden,web-vault}
-rwxr-xr-x 1 user user 53602704 Nov  5 23:57 output/vaultwarden
drwxr-xr-x 8 user user     4096 Nov  3 15:23 output/web-vault
```

To pull and extract an image for another platform:

* ARMv6: `./docker-image-extract -p linux/arm/v6 vaultwarden/server:latest-alpine`
* ARMv7: `./docker-image-extract -p linux/arm/v7 vaultwarden/server:latest-alpine`
* ARMv8 / AArch64: `./docker-image-extract -p linux/arm64 vaultwarden/server:latest-alpine`
