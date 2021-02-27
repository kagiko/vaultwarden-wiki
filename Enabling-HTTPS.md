For proper operation of bitwarden_rs, enabling [HTTPS](https://en.wikipedia.org/wiki/HTTPS) is pretty much required nowadays, since the Bitwarden web vault uses [web crypto APIs](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto) that most browsers only make available in HTTPS contexts.

There are a few ways you can enable HTTPS:

* (Recommended) Put bitwarden_rs behind a [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy) that handles HTTPS connections on behalf of bitwarden_rs.
* (Not recommended) Enable the HTTPS functionality built into bitwarden_rs (via the [Rocket](https://rocket.rs/) web framework). Rocket's HTTPS implementation is relatively immature and limited. This method also does not support [[WebSocket notifications|Enabling-WebSocket-notifications]].

Refer to the [Enabling HTTPS](#enabling-https) section for more details on these options.

For an HTTPS server to work, it also needs an SSL/TLS certificate, so you'll need to decide how to obtain this. Again, there are a few options:

* (Recommended) Get [Let's Encrypt](https://letsencrypt.org/) certificates using an [ACME client](https://letsencrypt.org/docs/client-options/). Some reverse proxies (e.g., [Caddy](https://caddyserver.com/)) also have built-in support for obtaining certs using the ACME protocol.
* (Recommended) If you trust [Cloudflare](https://www.cloudflare.com/) to proxy your traffic, you can let them handle issuance of your SSL/TLS certs. Note that the upstream Bitwarden web vault (https://vault.bitwarden.com/) runs behind Cloudflare.
* (Not recommended) [[Set up a private CA|Private-CA-and-self-signed-certs-that-work-with-Chrome]] and issue your own (self-signed) certificates. There are various pitfalls and inconveniences associated with this, so consider yourself warned.

Refer to the [Getting SSL/TLS certificates](#getting-ssltls-certificates) section for more details on these options.

## Enabling HTTPS

### Via a reverse proxy

There are quite a few reverse proxies in common use; some example configurations can be found at [[Proxy examples|Proxy-examples]]. If you aren't familiar with reverse proxies and have no preference, consider [Caddy](https://caddyserver.com/) first, since it has built-in support for obtaining Let's Encrypt certs.

### Via Rocket

:warning: This method is not recommended.

To enable HTTPS in `bitwarden_rs` itself, set the `ROCKET_TLS` environment variable, which has the following format:
```
ROCKET_TLS={certs="/path/to/certs.pem",key="/path/to/key.pem"}
```
where:
* `certs` is the path to the SSL/TLS certificate chain in PEM format.
* `key` is the path to the SSL/TLS certificate's corresponding private key file (in PEM format).

Notes:
* The file name _extensions_ used in the `ROCKET_TLS` line do not necessarily have to be `.pem` as in the example, and some places may issue certificates using other extension like `.crt` for the certificate and `.key` for the private key. It's the file _format_ that must be PEM, i.e. base64-encoded. Since the PEM format is openssl's default you can therefore simply rename .cert, .cer, .crt and .key files to .pem and vice versa or - as an alternative - use .crt or .key as file extensions in the `ROCKET_TLS` line.
* Use an RSA cert/key. Rocket is currently unable to handle an ECC cert/key, and outputs a misleading error message like

  > `[ERROR] environment variable ROCKET_TLS={certs="/ssl/ecdsa.crt",key="/ssl/ecdsa.key"} could not be parsed`

  (There's nothing wrong with the format of the environment variable itself; it's the cert/key contents that Rocket can't parse.)
* If running under Docker, remember that bitwarden_rs will be parsing the `ROCKET_TLS` value when running inside the container, so make sure the `certs` and `key` paths are how they would appear inside the container (which may be different from the paths on the Docker host system).

```sh
docker run -d --name bitwarden \
  -e ROCKET_TLS='{certs="/ssl/certs.pem",key="/ssl/key.pem"}' \
  -v /ssl/keys/:/ssl/ \
  -v /bw-data/:/data/ \
  -p 443:80 \
  bitwardenrs/server:latest
```

You need to mount ssl files (-v argument) and you need to forward appropriate port (-p argument), usually port 443 for HTTPS connections. If you choose a different port number than 443 like for example 3456, remember to explicitly provide that port number when you connect to the service, example: `https://bitwarden.local:3456`.

:warning: Make sure that your certificate file includes the full chain of trust. In the case of certbot, this means using `fullchain.pem` instead of `cert.pem`. The full chain should include two certs: the leaf cert (same as what's in `cert.pem`), followed by an R3 or E1 [intermediate cert](https://letsencrypt.org/certificates/#intermediate-certificates). For example, Android by default does not include any Let's Encrypt intermediate certs in their system trust store, so the Android client will likely fail to connect if you don't provide the full chain.

Software used for getting certs often use symlinks. If that is the case, both locations need to be accessible to the docker container.

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

#### Check if certificate is valid
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
The start of the output should look something like this (when using a Let's Encrypt cert):
```
CONNECTED(00000003)
depth=2 O = Digital Signature Trust Co., CN = DST Root CA X3
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = R3
verify return:1
depth=0 CN = vault.domain.com
verify return:1
```

Verify that there are 3 different depths (notice it starts at 0).
A bit further in the output you should see the base64-encoded certificates from Let's Encrypt itself.

## Getting SSL/TLS certificates

### Via Let's Encrypt

[Let's Encrypt](https://letsencrypt.org/) issues SSL/TLS certificates for free.

For this to work, your bitwarden_rs instance must have a DNS name (i.e., you can't simply use an IP address). Let's Encrypt is easier to set up if your bitwarden_rs is reachable on the public Internet, but even if your instance is private (i.e., only reachable on your LAN), it's still possible to get Let's Encrypt certs via [DNS challenge](Running-a-private-bitwarden_rs-instance-with-Let's-Encrypt-certs).

If you already own or control a domain, then just add a DNS name for the IP address of your bitwarden_rs instance. If you don't, you can either buy a domain name, try getting one for free at [Freenom](https://www.freenom.com/), or use a service like [Duck DNS](https://www.duckdns.org/) to get a name under an existing domain (e.g., `my-bitwarden.duckdns.org`).

Once you have a DNS name for your instance, use an [ACME client](https://letsencrypt.org/docs/client-options/) to get certs for your DNS name. [Certbot](https://certbot.eff.org/) and [acme.sh](https://github.com/acmesh-official/acme.sh) are two of the most popular standalone clients. Some reverse proxies like [Caddy](https://caddyserver.com/) also have built-in ACME clients.

### Via Cloudflare

[Cloudflare](https://www.cloudflare.com/) provides free service for individuals. If you trust them to proxy your traffic and serve as your DNS provider, you can let them handle issuance of your SSL/TLS certs as well.

Once you've enrolled your domain and added a DNS record for your bitwarden_rs instance, log into the Cloudflare dashboard and select `SSL/TLS`, then `Origin Server`. Generate an origin certificate (you can select a validity period up to 15 years) and configure bitwarden_rs to use it. If you've selected the 15-year validity period, you won't have to renew this origin certificate for the foreseeable future.

Note that the origin certificate is used only to secure communications between Cloudflare and bitwarden_rs. Cloudflare will automatically handle issuance and renewal of the certificate used for communicating between clients and Cloudflare.

Also, if you're using the Rocket HTTPS server built into bitwarden_rs, make sure to select `RSA` as the private key type for the origin certificate, as Rocket doesn't currently support ECC/ECDSA certs.
