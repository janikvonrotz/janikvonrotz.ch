---
title: Install s3cmd
date: 2014-04-10T07:48:14+00:00
author: Janik Vonrotz
slug: install-s3cmd
images:
  - /wp-content/uploads/2014/03/Amazon-AWS-Logo.png
categories:
  - Web server
tags:
  - aws
  - ubuntu
  - web server
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9548802](https://gist.github.com/9548802).*  

# Introduction

S3cmd is a command line tool for uploading, retrieving and managing data in Amazon S3 and other Cloud Storage Service Providers that use the S3 protocol
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [GnuPG](https://janikvonrotz.ch/2014/03/25/install-ubuntu-packages/)
* [Amazon AWS account](http://aws.amazon.com/)
* [Amazon IAM service user](https://console.aws.amazon.com/iam)
* [Amazon S3 bucket](https://console.aws.amazon.com/s3)

# Installation

Install the package with aptitude.

    sudo apt-get install s3cmd

Configure s3cmd.

    s3cmd --configure

Enter your Amazon AWS credentials.

    Access Key: [your access key]
    Secret Key: [your secret key]

Enter an encryption passwort for secure transmissions.

    Encryption password: [secure password]

Answert the next prompts as showed below.

    Path To GPG programm: [enter]
    Use HTTPS protocol [No]: [enter]
    HTTP Proxy server name: [depends on your network environment]
    Test access with supplied credentials: Y
    
If you'll get the following message.

    Error: Test failed: 4103 (AccessDenied): Access Denied
    
Try my policy.

```
{
  "Statement": [
    {
      "Action": [
        "s3:ListAllMyBuckets"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::*"
    },
    {
      "Action": [ 
          "s3:ListBucket", 
          "s3:PutObject",
          "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": [
          "arn:aws:s3:::[bucket name]", 
          "arn:aws:s3:::[bucket name]/*"
      ]
    }
  ]
}
```
    
# Source

[s3cmd website](http://s3tools.org/s3cmd)