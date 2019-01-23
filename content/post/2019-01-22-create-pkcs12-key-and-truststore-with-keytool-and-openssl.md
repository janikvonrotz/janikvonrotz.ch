---
title: "Create pkcs12 key- and truststore with keytool and openssl"
slug: create-pkcs12-key-and-truststore-with-keytool-and-openssl
date: 2019-01-22T14:05:15+01:00
categories:
 - Security
tags:
 - keytool
 - openssl
 - certificate
images:
 - /wp-content/uploads/2014/03/OpenSSL-Logo.png
---

In my [last post](https://janikvonrotz.ch/2019/01/21/create-a-certificate-authority-ca-and-sign-server-certificates-without-prompting-using-openssl/) I've showed you how to create a custom certificate authority and sign a server cert using openssl without user interaction.

For this post I assume that we want to set up a webservice that requires a [pkcs12](https://en.wikipedia.org/wiki/PKCS_12) keystore. Using openssl and the java keytool we are going to create a pkcs12 store and add our ca cert, server cert and server key. Further, we assume that the application also requires a truststore containing the ca cert only.

Make sure to walk through the last post before getting started.

Configure new environment variables.

```bash
APPLICATION_NAME=webservice
KEYSTORE_PASSWORD=password
TRUSTSTORE_PASSWORD=password
SERVER_CERT_PASSWORD=password
SERVER_CERT_CN=localhost
```

Create the pkcs12 store containing the server cert.

```bash
openssl pkcs12 -in ${SERVER_CERT_CN}_cert.pem -inkey ${SERVER_CERT_CN}_key.pem -passin pass:$SERVER_CERT_PASSWORD \
  -export -out $APPLICATION_NAME-keystore.pkcs12 -passout pass:$KEYSTORE_PASSWORD -name $SERVER_CERT_CN
```

Store manipulation is not possible with openssl. That is why the java keytool is required to import the ca cert into the store.

```bash
keytool -importcert -storetype PKCS12 -keystore $APPLICATION_NAME-keystore.pkcs12 \
  -storepass $KEYSTORE_PASSWORD -alias ca -file ca_cert.pem -noprompt
```

Show the content of keystore.

```bash
keytool -list -storetype PKCS12 -keystore $APPLICATION_NAME-keystore.pkcs12 \
  -storepass $KEYSTORE_PASSWORD
```

Openssl cannot create a pkcs12 store from cert without key. That is why we need to an empty truststore and use the keytool to import the ca cert.

Create an empty pkcs12 truststore with openssl.

```bash
openssl pkcs12 -in ca_cert.pem -nokeys \
  -export -out $APPLICATION_NAME-truststore.pkcs12 -passout pass:$TRUSTSTORE_PASSWORD
```

Then add the ca cert using the keytool.

```bash
keytool -importcert -storetype PKCS12 -keystore $APPLICATION_NAME-truststore.pkcs12 \
  -storepass $TRUSTSTORE_PASSWORD -alias ca -file ca_cert.pem -noprompt
```

Show the content of the truststore.

```bash
keytool -list -storetype PKCS12 -keystore $APPLICATION_NAME-truststore.pkcs12 \
  -storepass $TRUSTSTORE_PASSWORD
```

Not sure if it is a bug that openssl cannot create pkcs12 stores from certs without keys. Nonetheless, the two step workflow is a convenient solution. Openssl creates the initial pkcs12 store and the keytool manipulates the store as required.

**Update**: It seems you cannot import a certificate and its key with keytool. So you need to create the store with openssl in order to import the key.

Source: [Stackoverflow - How to import an existing x509 certificate and private key in Java keystore to use in SSL?](https://stackoverflow.com/questions/906402/how-to-import-an-existing-x509-certificate-and-private-key-in-java-keystore-to-u)