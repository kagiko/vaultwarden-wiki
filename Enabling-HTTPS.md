To enable HTTPS, you need to configure the `ROCKET_TLS`.

The values to the option must follow the format:
```
ROCKET_TLS={certs="/path/to/certs.pem",key="/path/to/key.pem"}
```
Where:
- certs: a path to a certificate chain in PEM format
- key: a path to a private key file in PEM format for the certificate in certs

```sh
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/certs.pem",key="/ssl/key.pem"}' \
  -v /ssl/keys/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  bitwardenrs/server:latest
```
Note that you need to mount ssl files and you need to forward appropriate port.

Due to what is likely a certificate validation bug in Android, you need to make sure that your certificate includes the full chain of trust. In the case of certbot, this means using `fullchain.pem` instead of `cert.pem`.

Softwares used for getting certs are often using symlinks. If that is the case, both locations need to be accessible to the docker container.

Example: [certbot](https://certbot.eff.org/) will create a folder that contains the needed `fullchain.pem` and `privkey.pem` files in `/etc/letsencrypt/live/mydomain/`

These files are symlinked to `../../archive/mydomain/privkey.pem`

So to use from bitwarden container:

```sh
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/live/mydomain/fullchain.pem",key="/ssl/live/mydomain/privkey.pem"}' \
  -v /etc/letsencrypt/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  bitwardenrs/server:latest
```