---
title: "Generate PEM key- and truststores With Puppet"
slug: generate-pem-key-and-truststores-with-puppet
date: 2019-03-07T10:23:04+01:00
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

This post is a follow-up of [Generate pkcs12 key- and truststores with Puppet](https://janikvonrotz.ch/2019/01/30/generate-pkcs12-key-and-truststores-with-puppet/).  
In this post we are going to create [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) key- and truststores with Puppet.

PEM files are base64 encoded [X.509](https://en.wikipedia.org/wiki/X.509) certificates. Enclosed between `-----BEGIN CERTIFICATE-----` and `-----END CERTIFICATE-----` multiple PEM files can be concatinated into key- and truststores. And that is exactly what we are going to do using a Puppet manifest.
<!--more-->

For the example use case multiple certificate files are required. Using the commands from the [Create a certificate authority and sign server certificates without prompting using openssl](https://janikvonrotz.ch/2019/01/21/create-a-certificate-authority-ca-and-sign-server-certificates-without-prompting-using-openssl/) post the following files must be provided:

* localhost_cert.pem
* localhost_key.pem
* example.com_cert.pem
* ca_cert.pem

The certificates will be concatinated into the following stores:

* webservice-keystore.pem
  * localhost - private key entry
* webservice-truststore.pem
  * ca - trusted cert entry
* custom-truststore.pem
  * example.com - trusted cert entry
  * ca - trusted cert entry

All the files are processed in `/var/tmp/certificates`.

The manifest file is defined as followed:

**modules/certbox/manifests/init.pp**

```rb
class certbox (

    String $host = 'localhost',
    String $cert_dir = '/var/tmp/certificates',

    String $server_ca_cert = "${cert_dir}/ca_cert.pem",
    String $server_cn = $host,
    String $server_cert = "${cert_dir}/${host}_cert.pem",
    String $server_key = "${cert_dir}/${host}_key.pem",
    String $server_key_pass = 'password',
    String $server_keystore = "${cert_dir}/webservice-keystore.pem",
    String $server_truststore = "${cert_dir}/webservice-truststore.pem",

    Array $custom_trust_entries = [
      {
        cn => 'example.com',
        cert => 'example.com_cert.pem'
      },
      {
        cn => 'ca',
        cert => 'ca_cert.pem'
      },
    ],
    String $custom_truststore = "${cert_dir}/custom-truststore.pem",

    String $owner = 'root',
    String $group = 'root',
    String $file_read_mode = 'a=,ug+r',

) {

  # component-wide defaulting of the exec path attribute
  Exec {
    path => ['/usr/bin', '/bin']
  }

  # create copy of sever cert, if file state changes notify keystore assemble task
  file { "${server_cert}.tmp":
    ensure => file,
    owner  => $owner,
    group  => $group,
    mode   => $file_read_mode,
    source => $server_cert,
    notify => Exec["certbox - create server keystore"],
  }

  # check if keystore file does not exist and notify creation task
  exec { "certbox - check if server keystore exists":
    onlyif => "test ! -f $server_keystore",
    command => 'echo "File does not exist."',
    notify => Exec["certbox - create server keystore"],
  }

  # create server keystore if temporary file state changed
  exec { "certbox - create server keystore":
    refreshonly => true,
    command     => "cat ${server_cert} > ${server_keystore}; cat ${server_key} >> ${server_keystore}",
  }

  # ensure server keystore file permissions
  file { $server_keystore:
    ensure => file,
    owner  => $owner,
    group  => $group,
    mode   => $file_read_mode,
  }

  file { "${server_ca_cert}.tmp":
    ensure => file,
    owner  => $owner,
    group  => $group,
    mode   => $file_read_mode,
    source => $server_ca_cert,
    notify => Exec["certbox - create server truststore"],
  }

  exec { "certbox - check if server truststore exists":
    onlyif => "test ! -f $server_truststore",
    command => 'echo "File does not exist."',
    notify => Exec["certbox - create server truststore"],
  }

  exec { "certbox - create server truststore":
    refreshonly => true,
    command     => "cat ${server_ca_cert} > ${server_truststore}",
  }

  file { $server_truststore:
    ensure => file,
    owner  => $owner,
    group  => $group,
    mode   => $file_read_mode,
    notify => Exec["certbox - create server truststore"],
  }

  # create copy for each ca truststore entry
  $custom_trust_entries.each | Integer $index, Hash $entry | {

    file { "${cert_dir}/${entry['cert']}.ca_trust.icam.tmp":
      ensure => file,
      owner  => $owner,
      group  => $group,
      mode   => $file_read_mode,
      source => "${cert_dir}/${entry['cert']}",
      notify => [
        Exec["certbox - cleanup ca truststore"],
        Exec["certbox - add pem ${entry['cn']} to ca truststore"]],
    }

    exec { "certbox - check if ca truststore for ${entry['cn']} exists":
      onlyif => "test ! -f $custom_truststore",
      command => 'echo "File does not exist."',
      notify => Exec["certbox - add pem ${entry['cn']} to ca truststore"],
    }
  }

  # reset ca truststore if temporary entry file state changed
  exec { "certbox - cleanup ca truststore":
    refreshonly => true,
    command     => "echo > ${custom_truststore}",
  }

  # add ca truststore entry if temporary entry file state changed
  $custom_trust_entries.each | Integer $index, Hash $entry | {

    exec { "certbox - add pem ${entry['cn']} to ca truststore":
      refreshonly => true,
      command     => "echo >> ${custom_truststore};cat ${cert_dir}/${entry['cert']} >> ${custom_truststore}",
    }
  }

  file { $custom_truststore:
    ensure  => file,
    owner   => $owner,
    group   => $group,
    mode    => $file_read_mode,
  }
}
```

This manifest checks if the source PEM files have changed or the target stores does not exist. If one of these cases apply the manifests creates the key- or truststore. Finally, it sets the file mode and ownership.

To apply the manifest with Puppet run the following command:

`sudo puppet apply --modulepath=modules/ -e "include certbox"`

To verify if the files have been assembled correctly use this keytool commands:

`sudo keytool -printcert -file /var/tmp/certificates/custom-truststore.pem`

Let me know if this post helped you to resolve a particular use case.