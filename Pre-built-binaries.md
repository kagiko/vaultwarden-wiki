bitwarden_rs doesn't currently provide standalone binaries as a separate download, but for platforms that have an Alpine-based Docker image available (currently x86-64 and ARMv7), you can extract standalone, statically-linked binaries from the official Docker images. Each Docker image also includes a matching web vault build (which is platform-independent).

## Extracting binaries with Docker installed

Assuming you want to extract binaries for the platform you're running on:
```
docker pull bitwardenrs/server:alpine
docker create --name bwrs bitwardenrs/server:alpine
docker cp bwrs:/bitwarden_rs .
docker cp bwrs:/web-vault .
docker rm bwrs
```

If you want binaries for a different platform (for example, you only have Docker installed on your x86-64 machine, but you want to run bitwarden_rs on a Raspberry Pi), add the `--platform` option to the `docker pull` command:
```
docker pull --platform linux/arm/v7 bitwardenrs/server:alpine
# Run remaining commands as above.
# Note that the `docker create` command may print a message like:
#   WARNING: The requested image's platform (linux/arm/v7) does not match the detected host platform (linux/amd64)
#   and no specific platform was requested
# This is expected, and isn't cause for concern.
```

## Extracting binaries without Docker installed

TODO