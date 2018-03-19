## Client-side SSL using self-signed Certificate Authority root

### Create the self-signed CA root
Organization & Common Name: Some human identifier for this server CA.

    openssl genrsa [-des3] -out ca.key 2048
    openssl req -new -x509 -days 365 -key ca.key -out ca.crt

### Create the Client Key and CSR
Organization & Common Name = Person name

    # Create the key
    openssl genrsa -des3 -out client.key 4096
    # Create a Certificate Signing Request
    openssl req -new -key client.key -out client.csr
    # Sign the certificate with the CA
    openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out client.crt

#### Convert Client Key to PKCS
So that it may be installed in most browsers.

    openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12

#### Convert Client Key to (combined) PEM
Combines `client.crt` and `client.key` into a single PEM file for programs using openssl.

    openssl pkcs12 -in client.p12 -out client.pem -clcerts
Or

    cat client.key > client.pem
    cat client.crt >> client.pem

### Enable client certificate verification on web server
So the web server will require a user certificate provided by the CA

nginx

    ssl_client_certificate /path/to/ca.crt;
    ssl_verify_client [on|optional];
    
Apache2

    SSLCACertificateFile /path/to/ca.pem
    SSLOptions +StdEnvVars #optional
    SSLVerifyClient require
