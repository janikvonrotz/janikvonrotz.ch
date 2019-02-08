---
title: "Generate pkcs12 key- and truststores with Puppet"
slug: generate-pkcs12-key-and-truststores-with-puppet
date: 2019-01-30T11:16:52+01:00
categories:
 - Software configuration management
tags:
 - puppet
 - deployment
 - keytool
 - openssl
 - certificate
images:
 - /images/Puppet Logo.svg
---

In [my last post](https://janikvonrotz.ch/2019/01/22/create-pkcs12-key-and-truststore-with-keytool-and-openssl/) I have showed how to generate pkcs12 key- and truststores using openssl and keytool.

For this post we assume that we want to automate the store assembling with Puppet. [Puppet](https://puppet.com/) is a configuration management tool that shares many ideas with [Ansible](https://www.ansible.com/). In the world of Puppet you define a [manifest file](https://puppet.com/docs/puppet/5.5/lang_summary.html#files) that describes a state of how a file, service or any type of resource should look like. Puppet applies these manifests and makes sure that the targeted system reaches the defined state.
<!--more-->

Using Puppets [exec resource](https://puppet.com/docs/puppet/5.3/types/exec.html) we are going to define key- and truststores using openssl and keytool.

Our manifest assure that the stores are assembled as defined. Puppet will be able to detect deltas and act accordingly.

For our use-case we assume we have generate multiple certificates under `/var/tmp/certificates`:

* localhost_cert.pem
* localhost_key.pem
* example.com_cert.pem
* example.com_key.pem
* ca_cert.pem

Our fictive webservice requires a key- and truststore containing the provided certificates and keys:

* webservice-keystore.pkcs12
  * ca - trusted cert entry
  * localhost - private key entry
  * example.com - private key entry
* webservice-truststore.pkcs12
  * ca - trusted cert entry

Puppet is installed on our system and we are ready to declare and apply our manifest file.

Our Puppet module will be called *certbox*. Modules must follow a strict naming and folder structure.

Below is a copy-and-paste definition for our Puppet module. Create the manifest file and folders as showed.

**./modules/certbox/init.pp**

```rb
class pki (

  String $certDir = "/var/tmp/certificates",

  String $caCert = "$certDir/ca_cert.pem",

  String $cn1 = "localhost",
  String $cert1 = "$certDir/${cn1}_cert.pem",
  String $key1 = "$certDir/${cn1}_key.pem",
  String $keyPassword1 = "password",

  String $cn2 = "adnvl044.zh.adnovum.ch",
  String $tmpKeystore2 = "$certDir/$cn2.pkcs12",
  String $cert2 = "$certDir/${cn2}_cert.pem",
  String $key2 = "$certDir/${cn2}_key.pem",
  String $keyPassword2 = "password",

  String $keystore = "$certDir/webservice-keystore.pkcs12",
  String $truststore = "$certDir/webservice-truststore.pkcs12",
  String $keystorePassword = "password",
  String $truststorePassword = "password",

  String $owner = "root",
  String $group = "root",
  String $fileReadMode = "a=,ug+r",

) {

  exec { "remove keystore for $cn1 if password changed or is empty":
    onlyif => "/bin/keytool -list -storetype PKCS12 -keystore $keystore -storepass $keystorePass | grep 'password was incorrect\\|file exists, but is empty'",
    command => "/bin/rm $keystore1",
  }

  exec { "create pkcs12 keystore for $cn1":
    onlyif => "/bin/keytool -list -keystore $keystore -storepass $keystorePassword | grep $(openssl x509 -noout -fingerprint -sha1 -in $cert1 | cut -f2 -d \"=\");test $? -eq 1",
    command => "/bin/openssl pkcs12  -export -in $cert1 -inkey $key1 -passin pass:$keyPassword1 -certfile $caCert -out $keystore -passout pass:$keystorePassword -name $cn1",
  }

  exec { "remove keystore for $cn2 if password changed or is empty":
    onlyif => "/bin/keytool -list -storetype PKCS12 -keystore $tmpKeystore2 -storepass $keystorePass | grep 'password was incorrect\\|file exists, but is empty'",
    command => "/bin/rm $tmpKeystore2",
  }

  exec { "create pkcs12 keystore for $cn2":
    onlyif => "/bin/keytool -list -keystore $tmpKeystore2 -storepass $keystorePassword | grep $(openssl x509 -noout -fingerprint -sha1 -in $cert2 | cut -f2 -d \"=\");test $? -eq 1",
    command => "/bin/openssl pkcs12 -in $cert2 -inkey $key2 -passin pass:$keyPassword2 -export -out $tmpKeystore2 -passout pass:$keystorePassword -name $cn2",
  }

  exec { "merge $cn2 into $cn1 keystore":
    onlyif => "/bin/keytool -list -keystore $keystore -storepass $keystorePassword | grep $(openssl x509 -noout -fingerprint -sha1 -in $cert2 | cut -f2 -d \"=\");test $? -eq 1",
    command => "/bin/keytool -importkeystore -storetype PKCS12 -destkeystore $keystore -deststorepass $keystorePassword -destkeypass $keystorePassword \
      -srckeystore $tmpKeystore2 -srcstoretype PKCS12 -srcstorepass $keystorePassword -alias $cn2 -noprompt",
  }

  file { $keystore:
    ensure => file,
    owner => $owner,
    group => $group,
    mode => $fileReadMode,
  }

  exec { "remove truststore for $cn1 if password changed":
    onlyif => "/bin/keytool -list -storetype PKCS12 -keystore $truststore -storepass $truststorePass | grep 'password was incorrect'",
    command => "/bin/rm $truststore",
  }

  exec { "create pkcs12 truststore":
    onlyif => "/bin/keytool -list -keystore $truststore -storepass $truststorePassword | grep $(openssl x509 -noout -fingerprint -sha1 -in $caCert | cut -f2 -d \"=\");test $? -eq 1",
    command => "/bin/keytool -importcert -storetype PKCS12 -keystore $truststore -storepass $truststorePassword -alias ca -file $caCert -noprompt",
  }

  file { $truststore:
    ensure => file,
    owner => $owner,
    group => $group,
    mode => $fileReadMode,
  }
}
```

**Edit 1:** Compare sha1 of cert to assert if import should be executed.  
**Edit 2:** On create keystore check not if store file already exist, but check if cert in keystore matches the sha1. The check should also act as expected if file does not exist.
**Edit 3:** Remove the create empty truststore task. If the truststore file does not exist, the keytool import command will create the file.
**Edit 4:** Added exec task that remove the store file if the password has changed.

The manifest is kept fairly simple. However, you might ask why we have to merge our second certificate from a pkcs12 store. The answer is a bit more complicated. The openssl and keytool utilities help generating and managing key material. They provide similar tasks, but differ heavy in specific features. Here are the most important differences:

* Openssl cannot manipulate existing pkcs12 stores.
* Openssl cannot create pkcs12 with multiple certificates and keys.
* Keytool cannot import certificates and keys into an existing pkcs12 store, however, it can import a pkcs12 store into an existing one.
* Keytool cannot create pkcs12 stores from a certificate and key.

Hope this helps to understand the design decisions made here much better.

To apply the manifest with Puppet run the following command:

`sudo puppet apply --modulepath=./modules/ -e "include certbox"`

At this point Puppet should create the key- and truststores.

If you ended up with an error or faced some other issues let me know!