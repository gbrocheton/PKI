### Client-side SSL using self-signed Certificate Authority root (~)

#### Create a self-signed root Certificate Authority
Organization & Common Name: Some human identifier for this server CA.

    # Create a self-signed certificate to use as certificate authority. Optionnal: Replace 2048 by 4096. Add -des3 option.
    openssl genrsa -out ca.key 2048
    openssl req -new -x509 -days 365 -key ca.key -out ca.crt

#### Create a host Key and a Certificate Signing Request
Subject name must match website name for webserver SSL certificate

    # Create the key. Optionnal: Replace 2048 by 4096. Add -des3 option.
    openssl genrsa -out client.key 2048
    # Create a Certificate Signing Request
    openssl req -new -key client.key -out client.csr
    # Sign the certificate with the CA. For web servers use the next command.
    openssl x509 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -out client.crt [-set_serial 10|-CAcreateserial]

    # Add Subject alternative name. Useful for web servers.
    openssl x509 -req -days 3650 -in host.csr -CA ca.crt -CAkey ca.key -out host.crt -extfile <(echo "subjectAltName=DNS:host.infra.lan,IP:10.0.0.10")
    #If it doesn't work, use:
    echo "subjectAltName=DNS:host.infra.lan,IP:10.0.0.10" > san.txt
    openssl x509 -req -days 3650 -in host.csr -CA ca.crt -CAkey ca.key -out host.crt -extfile san.txt

    # Tip: Check CSR information
    openssl req -text -noout -in private.csr

#### Convert host certificate to P12 and PEM
/!\ These formats contains the private key. They are useful for some applications.

    # Convert from CRT to P12
    openssl pkcs12 -export -in client.crt -inkey client.key -out client.p12 [-clcerts|-cacerts|-nocerts]
    # Convert from P12 to PEM. This merges 
    openssl pkcs12 -in client.p12 -out client.pem [-clcerts|-cacerts|-nocerts]
Following method might work too:

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
