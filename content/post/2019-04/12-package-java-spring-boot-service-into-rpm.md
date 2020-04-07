---
title: "Package Java Spring Boot service into rpm"
slug: package-java-spring-boot-service-into-rpm
date: 2019-04-12T11:38:15+02:00
categories:
 - Software configuration management
tags:
 - rpm
 - spring boot
 - systemd
 - fedora
 - best practice
 - guide
images:
 - /images/shipping container.jpg
---

This post is a follow-up to my last post [The final rpm packaging guide](/2019/03/20/the-final-rpm-packaging-guide/). What I did not cover in the "final" guide was how to deal with the common case of packaging a service. In this post we are going to build a simple java spring boot application and package it as a [systemd](https://www.linux.com/learn/understanding-and-using-systemd) service into an rpm.
<!--more-->

To get started I recommend to walk through the test setup section of [my last post](/2019/03/20/the-final-rpm-packaging-guide#test-setup).

# Requirements

To accomplish this tutorial a few dev tools must be installed.

[httpie](https://httpie.org) - Modern alternative to wget and curl.

`sudo yum install httpie`

[rpmbuild](https://rpm.org/) - Utility to build rpms.

`sudo yum install rpm-build`

[rpmdev](https://fedoraproject.org/wiki/Rpmdevtools) - Dev utilities to build rpms. 

`sudo yum install rpmdevtools`

[rpmlint](https://github.com/rpm-software-management/rpmlint) - Tool for checking common errors in rpm packages.

`sudo yum install rpmlint`

# Example Application

We assume that we want to deploy a spring boot application.

Lets create one from the spring boot getting started source.

Checkout the official getting started repository.

`cd ~ && git clone https://github.com/spring-guides/gs-spring-starter.git`

Navigate into the project.

`gs-spring-starter/initial`

Then build and run the jar file.

`./gradlew build && java -jar build/libs/gs-spring-starter-0.1.0.jar`

Check if you get a response from the spring boot service.

`http GET localhost:8080`

It should return:

```txt
HTTP/1.1 200 
Content-Length: 27
Content-Type: text/plain;charset=UTF-8
Date: Fri, 12 Apr 2019 07:28:37 GMT

Greetings from Spring Boot!
```

# Project Setup

Create the initial rpm folder structure.

`cd ~; mkdir rpmbuild; cd ~/rpmbuild; rpmdev-setuptree`

Copy the jar file into the source folder.

`cp ../gs-spring-starter/initial/build/libs/gs-spring-starter-0.1.0.jar SOURCES/`

Our example service will be managed by systemd and thus we also have to package a service file into the rpm.

Create the file `vim SOURCES/spring-starter.service` with the following content:

```txt
[Unit]
Description=Spring Starter
After=network-online.target

[Service]
Type=simple
WorkingDirectory=/var/opt/spring-starter
ExecStart=/usr/bin/java -jar /usr/local/spring-starter/gs-spring-boot.jar
Restart=on-abort
User=spring-starter
Group=spring-starter

[Install]
WantedBy=multi-user.target
```

And thats it.

# Write the Spec

The spec file defines the rpm build process and installation procedure.

Create a spec file for our spring starter application.

`vim SPECS/spring-starter.spec`

And set the following content:

```bash
##### HEADER SECTION #####

Name:           spring-starter
Version:        0.1.0
Release:        0
Summary:        Rpm package for Spring Starter

License:        ASL 2.0
URL:            https://spring.io
Source0:        gs-spring-boot-%{version}.jar
Source1:				%{name}.service

Requires:       shadow-utils,bash
BuildRequires:	systemd
%{?systemd_requires}

BuildArch:      noarch

%description
%{summary}

# disable debuginfo, which is useless on binary-only packages
%define debug_package %{nil}

# do not repack jar files
%define __jar_repack %{nil}

##### PREPARATION SECTION #####
%prep

# empty section

##### BUILD SECTION #####
%build

# empty section

##### PREINSTALL SECTION #####
%pre

# create Spring Starter service group
getent group spring-starter >/dev/null || groupadd -f -g 30000 -r spring-starter

# create Spring Starter service user
if ! getent passwd spring-starter >/dev/null ; then
    if ! getent passwd 30000 >/dev/null ; then
      useradd -r -u 30000 -g spring-starter -d /home/spring-starter -s /sbin/nologin -c "Spring Starter service account" spring-starter
    else
      useradd -r -g spring-starter -d /home/spring-starter -s /sbin/nologin -c "Spring Starter service account" spring-starter
    fi
fi
exit 0

##### INSTALL SECTION #####
%install

app_dir=%{buildroot}/usr/local/spring-starter
data_dir=%{buildroot}/var/opt/spring-starter
service_dir=%{buildroot}/%{_unitdir}

# cleanup build root
rm -rf %{buildroot}
mkdir -p  %{buildroot}

# create app folder
mkdir -p $app_dir

# create data folder
mkdir -p $data_dir

# create service folder
mkdir -p $service_dir

# copy all files
cp %{SOURCE0} $app_dir/gs-spring-boot.jar
cp %{SOURCE1} $service_dir

##### FILES SECTION #####
%files

# define default file attributes
%defattr(-,spring-starter,spring-starter,-)

# list of directories that are packaged
%dir /usr/local/spring-starter
%dir %attr(660, -, -) /var/opt/spring-starter

# list of files that are packaged
/usr/local/spring-starter/gs-spring-boot.jar
/usr/lib/systemd/system/%{name}.service

##### POST INSTALL SECTION #####
%post

# ensure Spring Starter service is enabled and running
%systemd_post %{name}.service
%{_bindir}/systemctl enable %{name}.service
%{_bindir}/systemctl start %{name}.service

##### UNINSTALL SECTION #####
%preun

# ensure Spring Starter service is disabled and stopped
%systemd_preun %{name}.service

%postun

case "$1" in
	0) # This is a package remove

		# remove app and data folders
		rm -rf /usr/local/spring-starter
		rm -rf /var/opt/spring-starter

		# remove Spring Starter service user and group
		userdel spring-starter
	;;
	1) # This is a package upgrade
		# do nothing
	;;
esac

# ensure Spring Starter service restartet if an upgrade is performed
%systemd_postun_with_restart %{name}.service

##### CHANGELOG SECTION #####
%changelog

* Wed Mar 20 2019 Janik vonRotz <contact@janikvonrotz.ch> - 0.1.0-0
- First spring-starter package
```

I wont go into details about the spec as I already did that in my last post. Not much has changed.
However, one thing worth to mention is the `%define __jar_repack %{nil}` definition. This options disables the compression for `.jar` files. If the jar file is compressed it becomes inexecutable.

Further, as you might have noticed there are systemd macros, which ensure the service is configured properly. In the post install section two additional commands have been added to enable and start the service. This was done because t the systemd macros do not enable or start service by default.

# Build

Build the rpm.

`rpmbuild -ba SPECS/spring-starter.spec`

Lint the rpm.

`rpmlint RPMS/noarch/spring-starter-0.1.0-0.noarch.rpm`

Rpmlint is very verbose, most warnings can be ignored.

Install the rpm.

`sudo yum install RPMS/noarch/spring-starter-0.1.0-0.noarch.rpm -y`

Check if the spring starter service is running.

`systemctl status spring-starter`

And have a look at the interface.

`http GET localhost:8080`

If the service responds you have accomplished the tutorial.

Uninstall the rpm.

`sudo yum remove spring-starter -y`

And thats it.

As usual feel free to ask any question in the comments.

See ya 😄

# Helpers

Output rpm scriptlets.  
`rpm -E %{_unitdir}`

# Sources

Sources that I used to build this tutorial.

[Fedora Packaging Guidelines - Systemd](https://docs.fedoraproject.org/en-US/packaging-guidelines/Scriptlets/#_systemd)

[Fedora Wiki - Packaging Systemd](https://fedoraproject.org/wiki/Packaging:Systemd)

[Github - Systemd Macros Source](https://github.com/systemd/systemd/blob/master/src/core/macros.systemd.in)

[Golinuxhub - How to execute a script at %pre, %post, %preun or %postun stage (spec file) while installing/upgrading an rpm](https://www.golinuxhub.com/2018/05/how-to-execute-script-at-pre-post-preun-postun-spec-file-rpm.html)