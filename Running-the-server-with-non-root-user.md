The root user inside the container is already pretty limited in what it can do, so the default setup should be secure enough. However if you wish to go the extra mile to avoid using root even in container, here's how you can do that:

  1. Create a data folder that's owned by non-root user, so you can use that user to write persistent data. Get the user `id`. In linux you can run `stat <folder_name>` to get/verify the owner ID.
  2. When you run the container, you need to provide the user ID as one of the parameters. Note that this needs to be in the numeric form and not the username, because docker would try to find such user-defined inside the image, which would likely not be there or it would have different ID than your local user and hence wouldn't be able to write the persistent data. This can be done with the `--user` parameter.
  3. bitwarden_rs listens on port `80` inside the container by default, this [won't work with non-root user](https://www.w3.org/Daemon/User/Installation/PrivilegedPorts.html), because regular users aren't allowed to open port below `1024`. To overcome this, you need to configure server to listen on a different port, you can use `ROCKET_PORT` to do that.

Here's sample docker run, that uses user with id `1000` and with the port redirection configured, so that inside container the service is listening on port `8080` and docker translates that to external (host) port `80`:

```sh
docker run -d --name bitwarden \
  --user 1000 \
  -e ROCKET_PORT=8080 \
  -v /bw-data/:/data/ \
  -p 80:8080 \
  bitwardenrs/server:latest