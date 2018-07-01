---
id: 1871
title: Install s3cmd
date: 2014-04-10T07:48:14+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1871
permalink: /2014/04/10/install-s3cmd/
dsq_thread_id:
  - "2600602242"
image: /wp-content/uploads/2014/03/Amazon-AWS-Logo.png
categories:
  - Ubuntu Server
tags:
  - access
  - amazon
  - cmd
  - remote
  - s3
  - service
  - tool
  - ubuntu
  - web
---
*This post is part of my [Your own Virtual Private Server hosting solution](http://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
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

[code]
{
  &quot;Statement&quot;: [
    {
      &quot;Action&quot;: [
        &quot;s3:ListAllMyBuckets&quot;
      ],
      &quot;Effect&quot;: &quot;Allow&quot;,
      &quot;Resource&quot;: &quot;arn:aws:s3:::*&quot;
    },
    {
      &quot;Action&quot;: [ 
          &quot;s3:ListBucket&quot;, 
          &quot;s3:PutObject&quot;,
          &quot;s3:GetObject&quot;
      ],
      &quot;Effect&quot;: &quot;Allow&quot;,
      &quot;Resource&quot;: [
          &quot;arn:aws:s3:::[bucket name]&quot;, 
          &quot;arn:aws:s3:::[bucket name]/*&quot;
      ]
    }
  ]
}
[/code]
    
# Source

[s3cmd website](http://s3tools.org/s3cmd)