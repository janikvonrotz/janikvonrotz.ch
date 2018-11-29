---
title: "Find certificate files that will expire soon and create a csr"
slug: find-certificate-files-that-will-expire-soon-and-create-a-csr
date: 2018-11-29T13:44:51+01:00
categories:
 - Scripting
tags:
 - bash
 - openssl
images:
 - /wp-content/uploads/2014/03/OpenSSL-Logo.png
---

The certificate expiration period should be kept as short as possible in a public key infrastructure. But the cost of resigning certificates must not be too high. This trade off causes a lot of problems. Every now and then a certificate expires without anybody noticing it or the same certificate is used for 10 years, which is obviously a security risk. In order to avoid this problem you either use [Let’s Encrypt](https://letsencrypt.org/) or another fully automated certificate management system. If this is not available you must know at least which certificates are going to expire soon.
<!--more-->

In my case I had a project with multiple certificate and dynamically built key stores. I had to find the certificates in the project folder structure that expire soon and need to be resigned. In order to automate this process I've built the following bash script.

```sh
# look for certificates that will expire before this date
maxage="2018-12-31"

# create numeric date from max age
intmaxage=$(date -d $maxage +%s)

# search for all pem files in current folder
for certfile in $(find ./ -name *.pem); do

    # filter files by certificates
    if [[ "$certfile" == *certificate.pem ]]
    then

        # extract the not after date string
        noafter=$(openssl x509 -in $certfile -text -noout | grep 'Not After :' | cut -d':' -f2- | sed 's/ //')

        # convert it to a date value
        date=$(date --date="$noafter" "+%b %d %H:%M:%S %Y GMT")

        # convert date value to a numeric date
        intdate=$(date --date="$date" +%s)

        # set the key file
        keyfile=$(echo $certfile | sed 's/certificate.pem/key.pem/')

        # create csr file variable
        csrfile=$(basename $certfile | sed 's/certificate.pem/.csr/')

        # create new csr
        openssl req -out ~/$csrfile -key $keyfile -new

        # check if certificate expires before the max age
        if [[ $intdate -le $intmaxage ]]
        then

            # confirm the creation of the csr and provide meta information
            echo "A csr file: $csrfile"
            echo "for the certificate: $certfile"
            echo "with key file: $keyfile"
            echo "has been create as it will expire soon at: $date"
            echo ""
        fi
    fi
done
```