To enable HTTPS, you need to configure the `ROCKET_TLS`.

The values to the option must follow the format:
```
ROCKET_TLS={certs="/path/to/certs.pem",key="/path/to/key.pem"}
```
Where:
* certs: a path to a certificate chain in PEM format
* key: a path to a private key file in PEM format for the certificate in certs

Note:
* The file name _extensions_ used in the ROCKET_TLS line do not necessarily have to be PEM. Important is the underlying file _format_ which needs to be PEM, i.e. base64-coded. Since the PEM format is openssl's default you can therefore simply rename .cert, .cer, .crt and .key files to .pem and vice versa or - as an alternative - use different file extensions like .crt or .key in the ROCKET_TLS line.

```sh
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/certs.pem",key="/ssl/key.pem"}' \
  -v /ssl/keys/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  bitwardenrs/server:latest
```

You need to mount ssl files (-v argument) and you need to forward appropriate port (-p argument), usually 443 for HTTPS connections. If you choose a different port number than 443 like for example 3456, remember to explicitly provide that port number when you connect to the service, example: `https://bitwarden.local:3456`.

For further information on how to set up and use a private CA on your local system refer to [this chapter of the wiki.](https://github.com/dani-garcia/bitwarden_rs/wiki/Private-CA-and-self-signed-certs-that-work-with-Chrome)

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