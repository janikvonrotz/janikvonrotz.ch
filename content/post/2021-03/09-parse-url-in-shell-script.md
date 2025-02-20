---
title: "Parse URL in shell script"
slug: parse-url-in-shell-script
date: 2021-03-09T16:06:27+01:00
categories:
 - Scripting
tags:
 - url
 - parse
 - bash
images:
 - /images/Bash-logo.png
---
Use the bash script below to extract any segment of an URL. For example you can use it like this `parse-url https://www.example.com subdomain,proto`  and get the subdomain and protocol of the url as reponse.

<!--more-->

**/usr/local/bin/parse-url**

```bash
#!/bin/bash

set -e

# Get script name
SCRIPT=$(basename "$0")

# Display Help
Help() {
    echo
    echo "$SCRIPT"
    echo
    echo "Description: Filter url for segments."
    echo "Syntax: $SCRIPT <url> [url|proto|user|pass|host|domain|subdomain|port|path]"
    echo "Example: $SCRIPT https://www.example.com subdomain,domain"
    echo
}

# Show help and exit
if [[ $1 == 'help' ]]; then
    Help
    exit
fi

# check params
[[ -z "$2" ]] && { echo "Parameter filter is empty." ; exit 1; }

# get protocol
proto="$(echo $1 | grep :// | sed -e's,^\(.*://\).*,\1,g')"

# remove the protocol
url="$(echo ${1/$proto/})"

# extract the user (if any)
userpass="$(echo $url | grep @ | cut -d@ -f1)"
pass="$(echo $userpass | grep : | cut -d: -f2)"
if [ -n "$pass" ]; then
user="$(echo $userpass | grep : | cut -d: -f1)"
else
    user=$userpass
fi

# extract the host
host="$(echo ${url/$user@/} | cut -d/ -f1)"

# by request - try to extract the port
port="$(echo $host | sed -e 's,^.*:,:,g' -e 's,.*:\([0-9]*\).*,\1,g' -e 's,[^0-9],,g')"

# extract the path (if any)
path="$(echo $url | grep / | cut -d/ -f2-)"

# extract domain and subdomain
levels=$(echo $host | grep -o "\." | wc -l)
if [ $levels -eq "1" ]; then 
    domain=$host
fi
if [ $levels -eq "2" ]; then 
    domain=$(echo $host | awk -F"." '{print $2 "." $3}')
    subdomain=$(echo $host | awk -F"." '{print $1}')
fi

for f in $(echo $2 | sed "s/,/ /g")
do
    case $f in
        url)
            echo $url
            ;;
        proto)
            echo $proto
            ;;
        user)
            echo $user
            ;;
        pass)
            echo $pass
            ;;
        host)
            echo $host
            ;;
        domain)
            echo $domain
            ;;
        subdomain)
            echo $subdomain
            ;;
        port)
            echo $port
            ;;
        path)
            echo $path
            ;;
    esac
done
```

Ensure that the script is executable `chmod +x /usr/local/bin/parse-url`.