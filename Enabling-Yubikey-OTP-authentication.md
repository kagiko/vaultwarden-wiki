To enable YubiKey authentication, you must set the `YUBICO_CLIENT_ID` and `YUBICO_SECRET_KEY` env variables.

If `YUBICO_SERVER` is not specified, it will use the default YubiCloud servers. You can generate `YUBICO_CLIENT_ID` and `YUBICO_SECRET_KEY` for the default YubiCloud [here](https://upgrade.yubico.com/getapikey/).

Notes: 
* In order to generate API keys or use a YubiKey with an OTP server, it must be registered. After configuring your key in the [YubiKey Personalization Tool](https://www.yubico.com/products/services-software/personalization-tools/use/), you can register it with the default servers [here](https://upload.yubico.com/).
* aarch64 builds of the server version 1.6.0 or older do not support Yubikey functionality due to upstream issues - see [#262](https://github.com/dani-garcia/vaultwarden/issues/262). 

```sh
docker run -d --name bitwarden \
  -e YUBICO_CLIENT_ID=12345 \
  -e YUBICO_SECRET_KEY=ABCDEABCDEABCDEABCDE= \
  -v /bw-data/:/data/ \
  -p 80:80 \
  vaultwarden/server:latest
```
