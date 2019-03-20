---
title: "The final rpm packaging guide"
slug: the-final-rpm-packaging-guide
date: 2019-03-20T14:22:16+01:00
categories:
 - Blog
tags:
 - rpm
 - apache
 - artemis
 - fedora
 - best practice
 - guide
images:
 - /images/airplane engine.jpg
---

A customer required that every component of a software project I worked for is delivered as rpm.

This was problematic for [Apache ActiveMQ Artemis](http://activemq.apache.org/artemis/) as there is rpm provided.

But, the maintainer publish a binary tarball and that is all we need to build an rpm.

So in this post I am going to show how to build an rpm from a tarball and explain the rpm build process in detail.
<!--more-->

# Introduction

I highly recommend to read the [RPM Packaging Guide](https://rpm-packaging-guide.github.io/) to get a basic understanding of the rpm build and packaging process.

# Requirements

Lets get started by installing some dev tools.

[httpie](https://httpie.org) - Modern alternative to wget and curl.

`sudo yum install httpie`

[rpmbuild](https://rpm.org/) - Utility to build rpms.

`sudo yum install rpm-build`

[rpmdev](https://fedoraproject.org/wiki/Rpmdevtools) - Dev utilities to build rpms. 

`sudo yum install rpmdevtools`

[rpmlint](https://github.com/rpm-software-management/rpmlint) - Tool for checking common errors in rpm packages.

`sudo yum install rpmlint`

# Test Setup

Before we setup the dev chain for the apache Artemis package, let's get familiar with an hello world example.

Create a working dir wherever you would like.

`mkdir rpmbuild`

Create a symbolic to the `rpmbuild` folder from the home folder.

`ln -s /path/to/rpmbuild ~/rpmbuild`

Add default directories to build the RPM

`cd ~/rpmbuild; rpmdev-setuptree`

| Macro Name       | Name                    | Location               | Purpose |
|:-----------------|:------------------------|:-----------------------|:------- |
| `%_specdir`      | Specification directory | `~/rpmbuild/SPECS`     | RPM specifications (`.spec`) files |
| `%_sourcedir`    | Source directory        | `~/rpmbuild/SOURCES`   | Pristine source package (e.g. tarballs) and patches |
| `%_builddir`     | Build directory         | `~/rpmbuild/BUILD`     | Source files are unpacked and compiled in a subdirectory underneath this. |
| `%_buildrootdir` | Build root directory    | `~/rpmbuild/BUILDROOT` | Files are installed under here during the `%install` stage. |
| `%_rpmdir`       | Binary RPM directory    | `~/rpmbuild/RPMS`      | Binary rpms are created and stored under here. |
| `%_srcrpmdir`    | Source RPM directory    | `~/rpmbuild/SRPMS`     | Source rpms are created and stored here. |

Create spec file for the hello world example.

`touch  ~/rpmbuild/SPECS/hello-world.spec`

And set the following content.

```bash
Name:       hello-world
Version:    1
Release:    1
Summary:    Most simple RPM package
License:    FIXME

%description
This is my first RPM package, which does nothing.

%prep
# we have no source, so nothing here

%build
cat > hello-world.sh <<EOF
#!/usr/bin/bash
echo Hello world
EOF

%install
mkdir -p %{buildroot}/usr/bin/
install -m 755 hello-world.sh %{buildroot}/usr/bin/hello-world.sh

%files
/usr/bin/hello-world.sh

%changelog
# let skip this for now
```

Build the hello world rpm.

`cd ~/rpmbuild; rpmbuild -ba SPECS/hello-world.spec`

Install the rpm.

`sudo yum install RPMS/x86_64/hello-world-1-1.x86_64.rpm -y`

Run hello world.

`hello-world.sh`

If the installation was successful you should see now the output: `hello world`.

Uninstall the hello world rpm.

`sudo yum remove hello-world -y`

Now the hello world command must not be available.

I assume that the hello world spec file is easy to understand. The install section might be misleading, but I will cover that in the next chapter.

Lets move with the more complex case.

# Apache Artemis Spec File

Create the spec file for the apache Artemis.

`touch  ~/rpmbuild/SPECS/apache-artemis.spec`

Navigate into the sources folder.

`cd ~/rpmbuild/SOURCES`

And download the apache Artemis binary.

`http -d http://www.pirbot.com/mirrors/apache/activemq/activemq-artemis/2.6.4/apache-artemis-2.6.4-bin.tar.gz`

## Sections

The spec file has been split into multiple sections. Copy and paste the snippet for every section and read the notes carefully.

### Header

```bash
##### HEADER SECTION #####

Name:           apache-artemis
Version:        2.6.4
Release:        1
Summary:        Rpm package for Apache ActiveMQ Artemis

License:        ASL 2.0
URL:            http://activemq.apache.org/artemis/
Source0:        %{name}-%{version}-bin.tar.gz

Requires:       shadow-utils,bash

BuildArch:      noarch

%description
%{summary}

# disable debuginfo, which is useless on binary-only packages
%define debug_package %{nil}
```

In the header section metadata is defined for the build process and for the package itself.

### Build

```bash
##### PREPARATION SECTION #####
%prep

# unpack tarball
%setup -q

##### BUILD SECTION #####
%build

# empty section
```

We use a tarball source, so no build step apart from unpacking the file is required. The `%setup` is a macro that extracts the tarball.

### Install

```bash
##### PREINSTALL SECTION #####
%pre

# create Apache Artemis service group
getent group artemis >/dev/null || groupadd -f -g 30101 -r artemis

# create Apache Artemis service user
if ! getent passwd artemis >/dev/null ; then
    if ! getent passwd 30101 >/dev/null ; then
      useradd -r -u 30101 -g artemis -d /home/artemis -s /sbin/nologin -c "Apache Artemis service account" artemis
    else
      useradd -r -g artemis -d /home/artemis -s /sbin/nologin -c "Apache Artemis service account" artemis
    fi
fi
exit 0

##### INSTALL SECTION #####
%install

app_dir=%{buildroot}/opt/app/artemis
data_dir=%{buildroot}/opt/data/artemis

# cleanup build root
rm -rf %{buildroot}
mkdir -p  %{buildroot}

# create app folder
mkdir -p $app_dir

# create data folder
mkdir -p $data_dir

# copy all files
cp LICENSE $app_dir
cp README.html $app_dir
cp -R bin $app_dir
cp -R lib $app_dir
cp -R schema $app_dir
cp -R web $app_dir
```

The `%install` section of an rpm spec file is not run on rpm package installation.

The `%install` section is run during package creation to install the files that need to be packaged such that the rpmbuild process can package them up.

Here we create the Artemis service user and group and copy selected files from the build to buildroot folder.

### File

```bash
##### FILES SECTION #####
%files

# define default file attributes
%defattr(-,artemis,artemis,-)

# list of directories that are packaged
%dir /opt/app/artemis
/opt/app/artemis/bin/*
/opt/app/artemis/lib/*
/opt/app/artemis/schema/*
/opt/app/artemis/web/*
%dir %attr(660, -, -) /opt/data/artemis

# list of files that are packaged
%doc /opt/app/artemis/README.html
%license /opt/app/artemis/LICENSE
```

The `%files` section lists all the files and directories that the package contains. The uninstallation process for an rpm is simply the removal of all the packaged files.

### Uninstall

```bash
##### UNINSTALL SECTION #####
%postun

case "$1" in
	0) # This is a package remove

		# remove app and data folders
		rm -rf /opt/app/artemis
    	rm -rf /opt/data/artemis

		# remove Apache Artemis service user and group
  		userdel artemis
	;;
	1) # This is a package upgrade
  		# do nothing
	;;
esac
```

If additional work needs to be done before or after the files are removed the `%preun` and `%postun` scriptlets are available in the spec file for that work.

### Changelog

```bash
##### CHANGELOG SECTION #####
%changelog

* Wed Mar 20 2019 Janik vonRotz <contact@janikvonrotz.ch> - 2.6.4-1
- Init apache-artemis package
```

Every time you make change, that is, whenever version or release of the package is incremented, add a comment to the changelog section

## Full Spec File

The final spec file should look like this.

```bash
##### HEADER SECTION #####

Name:           apache-artemis
Version:        2.6.4
Release:        1
Summary:        Rpm package for Apache ActiveMQ Artemis

License:        ASL 2.0
URL:            http://activemq.apache.org/artemis/
Source0:        %{name}-%{version}-bin.tar.gz

Requires:       shadow-utils,bash

BuildArch:      x86_64

%description
%{summary}

# disable debuginfo, which is useless on binary-only packages
%define debug_package %{nil}

##### PREPARATION SECTION #####
%prep

# unpack tarball
%setup -q

##### BUILD SECTION #####
%build

# empty section

##### PREINSTALL SECTION #####
%pre

# create Apache Artemis service group
getent group artemis >/dev/null || groupadd -f -g 30101 -r artemis

# create Apache Artemis service user
if ! getent passwd artemis >/dev/null ; then
    if ! getent passwd 30101 >/dev/null ; then
      useradd -r -u 30101 -g artemis -d /home/artemis -s /sbin/nologin -c "Apache Artemis service account" artemis
    else
      useradd -r -g artemis -d /home/artemis -s /sbin/nologin -c "Apache Artemis service account" artemis
    fi
fi
exit 0

##### INSTALL SECTION #####
%install

app_dir=%{buildroot}/opt/app/artemis
data_dir=%{buildroot}/opt/data/artemis

# cleanup build root
rm -rf %{buildroot}
mkdir -p  %{buildroot}

# create app folder
mkdir -p $app_dir

# create data folder
mkdir -p $data_dir

# copy all files
cp LICENSE $app_dir
cp README.html $app_dir
cp -R bin $app_dir
cp -R lib $app_dir
cp -R schema $app_dir
cp -R web $app_dir

##### FILES SECTION #####
%files

# define default file attributes
%defattr(-,artemis,artemis,-)

# list of directories that are packaged
%dir /opt/app/artemis
/opt/app/artemis/bin/*
/opt/app/artemis/lib/*
/opt/app/artemis/schema/*
/opt/app/artemis/web/*
%dir %attr(660, -, -) /opt/data/artemis

# list of files that are packaged
%doc /opt/app/artemis/README.html
%license /opt/app/artemis/LICENSE

##### UNINSTALL SECTION #####
%postun

case "$1" in
	0) # This is a package remove

		# remove app and data folders
		rm -rf /opt/app/artemis
    	rm -rf /opt/data/artemis

		# remove Apache Artemis service user and group
  		userdel artemis
	;;
	1) # This is a package upgrade
  		# do nothing
	;;
esac

##### CHANGELOG SECTION #####
%changelog

* Wed Mar 20 2019 Janik vonRotz <contact@janikvonrotz.ch> - 2.6.4-1
- Init apache-artemis package

```

# Installation

Navigate to the rpm build directory

`cd ~/rpmbuild`

Build the rpm.

`rpmbuild -ba SPECS/apache-artemis.spec`

Lint the rpm.

`rpmlint RPMS/x86_64/apache-artemis-2.6.4-1.x86_64.rpm `

There will be thousands of error and warnings. Ignore them for now.

Install the rpm.

`sudo yum install RPMS/x86_64/apache-artemis-2.6.4-1.May.rpm  -y`

Create a broker instance.

`sudo /opt/app/artemis/bin/artemis create broker /opt/data/artemis/broker --user artemis --password artemis --require-login`

Start the Artemis instance.

`sudo "/opt/data/artemis/broker/bin/artemis-service" start`

Open the browser to [localhost:8161/console](localhost:8161/console) and check if you can login with `artemis:artemis`.

If everything worked you can stop the service.

`sudo "/opt/data/artemis/broker/bin/artemis-service" stop`

And check if there is really no Artemis service running.

`ps aux | grep artemis`

Finally remove the package.

`sudo yum remove apache-artemis -y`

# Disclaimer

Apache Artemis has been installed to `/opt/app/artemis` and the instance has been created in `/opt/data/artemis`. Note that this is not best practice. The [Filesystem Hierarchy Standard](http://www.pathname.com/fhs/) tells where to put what.

# Conclusion

That's it! Lets recap what we just learned.

* First we created the rpmbuild folder structure
* Then created an example spec file for an hello world script
* Next we prepared the assets for the Apache Artemis package
* We defined the spec file for the rpm
* Following-up we installed the rpm
* Then created, started, tested and killed an Artemis instance
* Finally we uninstalled the Apache Artemis package

This is also resembles the life cycle of an rpm service.

As usual if you have any questions, concerns or inputs feel free to make a comment.

# Source

Sources I relied on to build this tutorials:

[Fedora Packaging Guidelines](https://docs.fedoraproject.org/en-US/packaging-guidelines/)

[RPM Packaging Guide](https://rpm-packaging-guide.github.io/)

[superuser - Managing service accounts in an RPM spec](https://superuser.com/questions/168461/managing-service-accounts-in-an-rpm-spec)

[packagecloud - Building RPM packages with rpmbuild](https://blog.packagecloud.io/rpm/rpmbuild/packaging/2015/06/29/building-rpm-packages-with-rpmbuild/)