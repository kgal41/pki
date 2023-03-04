# Root CA
## Generate a private key for the root CA
Use OpenSSL to generate a private key for the root CA. The following command generates a 256-bit Ed25519 private key:

```shell
openssl genpkey -algorithm Ed25519 -out rootCA.key
```

## Create a root CA certificate
Use OpenSSL to create a root CA certificate using the private key generated. The following command generates a self-signed root CA certificate valid for 10 years:

```shell
openssl req -new -x509 -key rootCA.key -out rootCA.crt -days 3650
```

## Verify the root CA certificate
Verify that the root CA certificate is valid and can be used to issue other certificates:

```shell
openssl x509 -noout -text -in rootCA.crt
```

# Sub CA
## Generate a private key for the subCA
Use OpenSSL to generate a private key for the sub CA. The following command generates a 256-bit Ed25519 private key:

```shell
openssl genpkey -algorithm Ed25519 -out subCA.key
```

## Create a certificate signing request (CSR)
Use OpenSSL to create a CSR for the sub CA. The following command creates a CSR for the sub CA:

```shell
openssl req -new -key subCA.key -out subCA.csr -subj "/C=US/ST=CA/O=Example/CN=Sub-CA"
```

## Sign the CSR with the rootCA
Use OpenSSL to sign the CSR with the root CA. The following command signs the CSR with the root CA certificate "rootCA.crt" and private key "rootCA.key", and generates a sub CA certificate valid for 10 year:

```shell
openssl x509 -req -in subCA.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out subCA.crt -days 3650 -extensions v3_ca -extfile <(echo "[v3_ca]\nbasicConstraints=CA:TRUE\nkeyUsage=keyCertSign,cRLSign\n")
```

## Verify the sub CA certificate
Verify that the sub CA certificate is valid and signed by the root CA:

```shell
openssl x509 -noout -text -in subCA.crt
```

# Subdomain with Sub CA
## Generate a private key for the subdomain
Use OpenSSL to generate a private key for the subdomain. The following command generates a 256-bit Ed25519 private key:

```shell
openssl genpkey -algorithm Ed25519 -out subdomain.key
```

## Create a certificate signing request (CSR)
Use OpenSSL to create a CSR for the subdomain. The following command creates a CSR for the subdomain "app.example.com":

```shell
openssl req -new -key subdomain.key -out subdomain.csr -subj "/C=US/ST=CA/O=Example/CN=app.example.com"
```

## Sign the CSR with the sub CA
Use OpenSSL to sign the CSR with the sub-CA. The following command signs the CSR with the sub-CA certificate "subCA.crt" and private key "subCA.key", and generates a subdomain certificate valid for 1 year:

```shell
openssl x509 -req -in subdomain.csr -CA subCA.crt -CAkey subCA.key -CAcreateserial -out subdomain.crt -days 365 -extensions v3_req -extfile <(echo "[v3_req]\nsubjectAltName=DNS:app.example.com")
```

## Verify the subdomain certificate
Verify that the subdomain certificate is valid and signed by the sub CA:

```shell
openssl x509 -noout -text -in subdomain.crt
```
