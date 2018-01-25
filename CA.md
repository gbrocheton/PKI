# Client-side SSL
For excessively paranoid client authentication.

## Using self-signed certificate.
### Create a Certificate Authority root

Organization & Common Name: Some human identifier for this server CA.

    openssl genrsa -des3 -out ca.key 4096
    openssl req -new -x509 -days 365 -key ca.key -out ca.crt


### Create the Client Key and CSR
Organization & Common Name = Person name

    openssl genrsa -des3 -out client.key 4096
    openssl req -new -key client.key -out client.csr
    # self-signed
    openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt

#### Convert Client Key to PKCS
So that it may be installed in most browsers.

    openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12

#### Convert Client Key to (combined) PEM
Combines `client.crt` and `client.key` into a single PEM file for programs using openssl.

    openssl pkcs12 -in client.p12 -out client.pem -clcerts

### Install CA cert on nginx
So that the Web server knows to ask for (and validate) a user's Client Key against the internal CA certificate.

    ssl_client_certificate /path/to/ca.crt;
    ssl_verify_client optional; # or `on` if you require client key
