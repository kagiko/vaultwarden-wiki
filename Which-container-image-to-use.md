If you plan to use Docker or Podman to run `bitwarden_rs`, you'll notice there are several image versions available. In most common case, you're going to run this service on amd64 (x86) hardware and you want to use the latest code. In this case `bitwardenrs/server` is the image to use.

Some users might want to use `bitwarden_rs` on different HW or they rather run a specific version of the service. In this case there are multiple options.

## Running on different HW architecture

The most common architecture is amd64 - your PC or server probably uses this one, so if you're not sure, this is the most likely case. It gets a bit tricky if you use some single board computers like Raspberry Pi as it might depend on CPU, but also on OS used. Generally speaking you can check your architecture by running `docker version` - see the `OS/Arch` info.

Based on your architecture, you can use one of the following images:

### `bitwardenrs/server:latest`

The "default image", runs on amd64. (x86, 64bit)

### `bitwardenrs/server:alpine`

Alpine-based amd64 image, same as above but a little bit smaller

### `bitwardenrs/server:raspberry`

Armv7hf image that should run on Raspberry Pi 2 or newer and possibly on any other compatible boards. This image won't run on Raspberry Pi 1 or Raspberry Pi Zero as those use armv6 CPU.

### `bitwardenrs/server:armv6`

Armv6 image for Raspberry Pi 1 and Raspberry Pi Zero.

### `bitwardenrs/server:aarch64`

Aarch64 image, that should run on ARMv8 devices like Raspberry Pi 3 or possibly other ARMv8 based devices.

**Note** that this will also require aarch64 distribution installed on your device, so for example if you use Raspbian on your Raspberry Pi 3, you still need to use `bitwardenrs/server:raspberry` as Raspbian is armv7hf distribution.

## Running a specific version

Normally it should be okay to use the latest version of the image as we aim to provide stable images there. However if you prefer to run a specific version of the service and update to the next version on your own pace, we do have versioned releases available.

You can run specific version by running image tagged with version number - for example `bitwardenrs/server:1.7.0`. If you run your service on different architecture (see above) you can use version provided for your architecture - like `bitwardenrs/server:1.7.0-raspberry`

**Note** that we have no control over releases of the client applications and they often expect to have the latest API supported, so running older versions of `bitwarden_rs` might lead to client app misbehaving. (e.g. missing or broken functionality) The Vault being an exception as it is bundled with the image and thus the version shipped there is going to be updated together with the service. This is why we usually recommend running the latest image as it should generally be the most compatible version with the updated upstream apps.

## Reported compatibility table

Please add your details here, if you're running the image on a hardware that is not already in the table.

| Hardware used        | OS           | Docker architecture reported    | Image used          | Status | Notes |
|----------------------|--------------|---------------------------------|---------------------|--------|-------|
| Regular 64bit server | Ubuntu 18.04 | x86_64                          | `bitwardenrs/server` | OK     |       |
| O-Droid HC2          | Armbian      | arm7l (arm32)                   | `registry.lollipopcloud.solutions/arm32v7/bitwarden` (see notes) | OK | Unofficial image built from upstream sources ; `bitwardenrs/server:raspberry` is the official equivalent image |
| Raspberry Pi Zero W  | Raspbian (4.14.98+) | linux/arm (armv6l)       | `bitwardenrs/server:armv6` | OK |     |
| Raspberry Pi 3 B     | Raspbian (4.14.98-v7+) | linux/arm (armv7l)    | `bitwardenrs/server:raspberry` | OK |     |
| Synology     | DSM (DSM 6.2.1-23824 Update 6) | Docker-x64-17.05.0-0367 | `bitwardenrs/server:latest` | OK |     |