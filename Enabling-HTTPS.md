To enable HTTPS, you need to configure the `ROCKET_TLS`.

The values to the option must follow the format:
```
ROCKET_TLS={certs="/path/to/certs.pem",key="/path/to/key.pem"}
```
Where:
* certs: a path to a certificate chain in PEM format
* key: a path to a private key file in PEM format for the certificate in certs

Notes:
* The file name _extensions_ used in the `ROCKET_TLS` line do not necessarily have to be PEM as in the example. Important is the file _format_ that needs to be PEM, i.e. base64-coded. Since the PEM format is openssl's default you can therefore simply rename .cert, .cer, .crt and .key files to .pem and vice versa or - as an alternative - use .crt or .key as file extensions in the `ROCKET_TLS` line.
* Use an RSA cert/key. Rocket appears to be unable to handle an ECC cert/key, and outputs a misleading error message like

  > `[ERROR] environment variable ROCKET_TLS={certs="/ssl/ecdsa.crt",key="/ssl/ecdsa.key"} could not be parsed`

  (There's nothing wrong with the format of the environment variable itself; it's the cert/key contents that Rocket can't parse.)

```sh
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/certs.pem",key="/ssl/key.pem"}' \
  -v /ssl/keys/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  bitwardenrs/server:latest
```

You need to mount ssl files (-v argument) and you need to forward appropriate port (-p argument), usually port 443 for HTTPS connections. If you choose a different port number than 443 like for example 3456, remember to explicitly provide that port number when you connect to the service, example: `https://bitwarden.local:3456`.

For further information on how to set up and use a private CA on your local system refer to [this chapter of the wiki.](https://github.com/dani-garcia/bitwarden_rs/wiki/Private-CA-and-self-signed-certs-that-work-with-Chrome) If following that guide your ROCKET_TLS line could look like this: `-e ROCKET_TLS='{certs="/ssl/bitwarden.crt",key="/ssl/bitwarden.key"}' \`

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

### Check if certificate is valid
When your bitwarden_rs server is available to the outside world you can use https://comodosslstore.com/ssltools/ssl-checker.php to check if your SSL certificate is valid including the chain. Without the chain Android devices will fail to connect.

You can also use https://www.ssllabs.com/ssltest/analyze.html to check, but that one does not support custom ports. Also please remember to check the "Do not show the results on the boards" checkbox, else your system will be visible in the "Recently Seen" list.

If you run a local server which does not have a connection to the public internet you could use the openssl tools to verify your certificate.

Execute the following to verify if the certificate is installed with the chains.
Chaing vault.domain.com to your own domain name.
```bash
openssl s_client -showcerts -connect vault.domain.com:443

# or with a different port
openssl s_client -showcerts -connect vault.domain.com:7070
```
The start of the output should look something like this (Using a Let's Encrypt Certificate):
```
CONNECTED(00000003)
depth=2 O = Digital Signature Trust Co., CN = DST Root CA X3
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = Let's Encrypt Authority X3
verify return:1
depth=0 CN = vault.domain.com
verify return:1
```

Verify that there are 3 different depths (notice it starts at 0).
A bit further in the output you should see the base64 encoded certificates from Let's Encrypt it self.
