# Development setup to test SSO

SSO support for Vaultwarden is currently [in development](https://github.com/dani-garcia/vaultwarden/pull/3154). The following describes a docker-compose based setup for locally testing these changes.

> [!WARNING]
> **ONLY USE FOR TESTING SSO, SETUP IS INSECURE**

## Setup

- Checkout the SSO branch
- Create `docker-compose.yml` with the following contents:
~~~
services:
  vaultwarden:
    build: .
    environment:
      DOMAIN: "http://localhost:8000"
      I_REALLY_WANT_VOLATILE_STORAGE: "true"
      SSO_ENABLED: "true"
      SSO_CLIENT_ID: "client"
      SSO_CLIENT_SECRET: "clientsecret"
      SSO_AUTHORITY: "http://auth.test:8080/mock"
    ports:
      - 127.0.0.1:8000:80

  mock-oauth2:
    image: ghcr.io/navikt/mock-oauth2-server:0.5.10
    hostname: "auth.test"
    ports:
      - 127.0.0.1:8080:8080
~~~
- Add `auth.test` to your systems host file:
  `echo "127.0.0.1 auth.test" | sudo tee -a /etc/hosts`
- Build vaultwarden:
  `docker compose build`

## Testing

- Start the services: `docker compose up`
- Go to [http://localhost:8000/#/sso](http://localhost:8000/#/sso), enter any string as identifier, click "Log in".
- On the Mock Auth2 Server Sign-in-Page, enter any string for user/subject and add the email you want to test in the claims field like so:
  `{"email": "user@example.com"}`
- If everything went according to plan, you will be asked for a master password.