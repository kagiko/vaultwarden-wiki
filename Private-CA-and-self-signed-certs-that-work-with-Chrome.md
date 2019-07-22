To get bitwarden working properly with self-signed certificates, chrome needs the certificate to include the domain name in the alternative name field of the certificate.

Create a CA key:
`openssl genrsa -des3 -out myCA.key 2048`

Create a CA certificate:
`openssl req -x509 -new -nodes -key myCA.key -sha256 -days 3650 -out myCA.pem`

Create a bitwarden key:
`openssl genrsa -out bitwarden.key 2048`

Create the bitwarden certificate request file:
`openssl req -new -key bitwarden.key -out bitwarden.csr`

Create a text file `bitwarden.ext` with the following, change the domain names to your setup.
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = bitwarden.local
DNS.2 = www.bitwarden.local
```


Create the bitwarden certificate, signed from the root CA:

```
openssl x509 -req -in bitwarden.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out bitwarden.crt -days 1825 -sha256 -extfile bitwarden.ext
```

Add the root certificate and the bitwarden certificate to client computers.