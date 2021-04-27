vaultwarden doesn't currently provide standalone binaries as a separate download, but for platforms that have an Alpine-based Docker image available (currently x86-64 and ARMv7), you can extract standalone, statically-linked binaries from the official Docker images. Each Docker image also includes a matching web vault build (which is platform-independent).

## Extracting binaries with Docker installed

Assuming you want to extract binaries for the platform you're running on:
```
docker pull vaultwarden/server:alpine
docker create --name bwrs vaultwarden/server:alpine
docker cp bwrs:/vaultwarden .
docker cp bwrs:/web-vault .
docker rm bwrs
```

If you want binaries for a different platform (for example, you only have Docker installed on your x86-64 machine, but you want to run vaultwarden on a Raspberry Pi), add the `--platform` option to the `docker pull` command:
```
docker pull --platform linux/arm/v7 vaultwarden/server:alpine
# Run remaining commands as above.
# Note that the `docker create` command may print a message like:
#   WARNING: The requested image's platform (linux/arm/v7) does not match the detected host platform (linux/amd64)
#   and no specific platform was requested
# This is expected, and isn't cause for concern.
```

## Extracting binaries without Docker installed

If you can't or don't want to install Docker, you can use the [docker-image-extract](https://github.com/jjlin/docker-image-extract) script to pull and extract a Docker image. For example, to pull and extract the x86-64 image:
```
$ mkdir bwrs-image
$ cd bwrs-image
$ wget https://raw.githubusercontent.com/jjlin/docker-image-extract/main/docker-image-extract
$ chmod +x docker-image-extract
$ ./docker-image-extract vaultwarden/server:alpine
Getting API token...
Getting image manifest for vaultwarden/server:alpine...
Downloading layer 801bfaa63ef2094d770c809815b9e2b9c1194728e5e754ef7bc764030e140cea...
Extracting layer...
Downloading layer c6d331ed95271d8005dea195449ab4ef943017dc97ab134a4426faf441ae4fa6...
Extracting layer...
Downloading layer bfd9ec32f740ca8c86ccde057595d29a31eb093aafd7619fcdd4b956c7bf95e3...
Extracting layer...
Downloading layer e9bfb5d92e4629b1dcb4a13a470c90f51b9edde4e184d8520afc589728b8b675...
Extracting layer...
Downloading layer 5757963c858ce72bc4a1874f4971d326d21d2a844f03063a3c99e312150adf95...
Extracting layer...
Downloading layer f705bf64e4315fea1830cc137d1deda194e825da03bd7822e41ac52457bc83e7...
Extracting layer...
Downloading layer 909b5deb38cbce9f83598918bf7f38b7c2194d385456cf7ef15eff47f8a63108...
Extracting layer...
Downloading layer 8516f4cd818630cd60fa18254b072f8d9c3748bdb56f6e2527dc1c204e8e017c...
Extracting layer...
Image contents extracted into ./output.
$ ls -ld output/{vaultwarden,web-vault}
-rwx------ 1 user user 22054608 Feb  6 21:46 output/vaultwarden
drwx------ 8 user user     4096 Feb  6 21:46 output/web-vault/
```

If you want the ARMv7 image, you currently have to download it by digest.

Go to https://hub.docker.com/r/vaultwarden/server/tags?name=alpine and find the entry for the `alpine` tag.
Click the partial digest for the `linux/arm/v7` image:

![](https://i.imgur.com/T5WdwtS.png)

This should bring you to a page that shows the full digest:

![](https://i.imgur.com/Hsz8vJ4.png)

Copy the full digest, and replace the `docker-image-extract vaultwarden/server:alpine` command above with
`docker-image-extract vaultwarden/server:<full_digest>`.

For example:
```
$ ./docker-image-extract vaultwarden/server:sha256:ef129de113bec3409b6370c37a6e5573a1dacc051a3aae2a8a3339323ae63623
```