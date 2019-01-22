---
title: "Create a certificate authority and sign server certificates without prompting using openssl"
slug: create-a-certificate-authority-ca-and-sign-server-certificates-without-prompting-using-openssl
date: 2019-01-21T16:20:35+01:00
categories:
 - Blog
tags:
 - keytool
 - openssl
 - certificate
images:
 - /wp-content/uploads/2014/03/OpenSSL-Logo.png
---

Most of the times people want to get a certificate for the hostname *localhost*, [let's encrypt wrote a nice post](https://letsencrypt.org/docs/certificates-for-localhost/) about this, but sometimes people want a certificate for any hostname. And further, signed by a custom CA and if possible should the key material be generated without user interaction. In this post I have covered the less likely case.
<!--more-->

Below you'll find a list of bash commands. You can copy and paste all of them into a bash script or run each command at a time in the shell.

Set environment variables for key material configuration.

```bash
CA_CERT_PASSWORD=password
SERVER_CERT_PASSWORD=password
SERVER_CERT_CN=localhost
SERVER_ALT_NAME=localhost
```

Create the CA key.

```bash
openssl genrsa -des3 -passout pass:$CA_CERT_PASSWORD -out ca_key.pem 4096
```

Create the CA certificate.

```bash
openssl req -x509 -new -nodes -key ca_key.pem -passin pass:$CA_CERT_PASSWORD -sha256 -days 1825 -out ca_cert.pem \
  -subj "/C=CH/ST=Bern/L=Bern/O=AdNovum AG/CN=ca"
```

Create the server key.

```bash
openssl genrsa -des3 -passout pass:$SERVER_CERT_PASSWORD -out ${SERVER_CERT_CN}_key.pem 2048
```

Create the server certificate signing request.

```bash
openssl req -new -key ${SERVER_CERT_CN}_key.pem -passin pass:$SERVER_CERT_PASSWORD -out ${SERVER_CERT_CN}.csr \
  -subj "/CN=$SERVER_CERT_CN"
```

Create server certificate extension configuration.

```bash
cat > ./${SERVER_CERT_CN}.cnf <<EOT
authorityKeyIdentifier=keyid,issuer
keyUsage=digitalSignature
extendedKeyUsage=serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = $SERVER_ALT_NAME
EOT
```

Sign the server certificate signing request.

```bash
openssl x509 -req -in ${SERVER_CERT_CN}.csr -CA ca_cert.pem -CAkey ca_key.pem -passin pass:$CA_CERT_PASSWORD -CAcreateserial \
  -out ${SERVER_CERT_CN}_cert.pem -days 1825 -sha256 -extfile ${SERVER_CERT_CN}.cnf
```

To create additional server certificates use the snippet below.

```bash
CA_CERT_PASSWORD=password
SERVER_CERT_PASSWORD=password
SERVER_CERT_CN=adnvl044.zh.adnovum.ch
SERVER_ALT_NAME=adnvl044.zh.adnovum.ch

openssl genrsa -des3 -passout pass:$SERVER_CERT_PASSWORD -out ${SERVER_CERT_CN}_key.pem 2048
  
openssl req -new -key ${SERVER_CERT_CN}_key.pem -passin pass:$SERVER_CERT_PASSWORD -out ${SERVER_CERT_CN}.csr \
  -subj "/CN=$SERVER_CERT_CN"
  
cat > ./${SERVER_CERT_CN}.cnf <<EOT
authorityKeyIdentifier=keyid,issuer
keyUsage=digitalSignature
extendedKeyUsage=serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = $SERVER_ALT_NAME
EOT
 
openssl x509 -req -in ${SERVER_CERT_CN}.csr -CA ca_cert.pem -CAkey ca_key.pem -passin pass:$CA_CERT_PASSWORD -CAcreateserial \
  -out ${SERVER_CERT_CN}_cert.pem -days 1825 -sha256 -extfile ${SERVER_CERT_CN}.cnf
```