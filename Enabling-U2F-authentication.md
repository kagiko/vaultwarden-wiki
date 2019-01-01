To enable U2F authentication, you must be serving bitwarden_rs from an HTTPS domain with a valid certificate (Either using the included
HTTPS options or with a reverse proxy). We recommend using a free certificate from Let's Encrypt.

After that, you need to set the `DOMAIN` environment variable to the same address from where bitwarden_rs is being served:

```sh
docker run -d --name bitwarden \
  -e DOMAIN=https://bw.domain.tld \
  -v /bw-data/:/data/ \
  -p 80:80 \
  mprasil/bitwarden:latest
```

Note that the value has to include the `https://` and it may include a port at the end (in the format of `https://bw.domain.tld:port`) when not using `443`.