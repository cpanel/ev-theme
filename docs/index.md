[![testsuite](https://github.com/cpanel/elevate/actions/workflows/testsuite.yml/badge.svg?branch=main)](https://github.com/cpanel/elevate/actions/workflows/testsuite.yml)

# Welcome to the cPanel Elevate project.

## Goal

This project provides a script to upgrade an existing `cPanel&WHM` CentOS 7 server installation to AlmaLinux 8.

## Introduction

- Issues can be reported here: [https://github.com/cpanel/elevate/issues](https://github.com/cpanel/elevate/issues)
- Pull requests are welcome: [https://github.com/cpanel/elevate/pulls](https://github.com/cpanel/elevate/pulls)

This project builds on the [Alma Linux Elevate](https://wiki.almalinux.org/elevate/ELevate-quickstart-guide.html) project, which leans heavily on the [LEAPP Project](https://leapp.readthedocs.io/en/latest/) created for in-place upgrades of RedHat based systems.

The Alma Linux Elevate project is very effective at upgrading the distro packages from [CentOS 7](https://www.centos.org/) to [AlmaLinux 8](https://almalinux.org/). However if you attempt to do this on a CentOS 7 based [cPanel install](https://www.cpanel.net/), you will end up with a broken system.

## Before updating

Before updating, please check that you met all the pre requirements:

* You will need to have console access available to your machine
* You should back up your server before attempting this upgrade
* Ensure your server is up to date: `yum update`
* Ensure you are using the last stable version of cPanel&WHM
* Use a version of MySQL/MariaDB compliant with Alamlinux 8.

**The cPanel elevate project does not back up before upgrading**

## Download the elevate-script

* You can download a copy of the script to run on your cPanel server via:

```bash
wget -O /scripts/elevate-cpanel \
    https://raw.githubusercontent.com/cpanel/elevate/stable/elevate-script ;
chmod 700 /scripts/elevate-cpanel
```

## Usage

```bash
# Read the help (and risks mentionned in this documentation)
/scripts/elevate-cpanel --help

# Check if your server is ready for elevation (dry run mode)
/scripts/elevate-cpanel --check

# Start the migration
/scripts/elevate-cpanel --start

... # expect multiple reboots (~30 min)

# Check the current status
/scripts/elevate-cpanel --status

# Monitor the elevation log
/scripts/elevate-cpanel --log

# In case of errors, once fixed you can continue the migration process
/scripts/elevate-cpanel --continue
```

## Some of the problems you might find include:

* x86_64 RPMs not in the primary CentOS repos are upgraded.
  * `rpm -qa|grep el7`
* EA4 RPMs are incorrect
  * EA4 provides different dependencies and linkage on C7/A8
* cPanel binaries (cpanelsync) are invalid.
* 3rdparty repo packages are not upgraded (imunify 360, epel, ...).
* Manually installed Perl XS (arch) CPAN installs invalid.
* Manually installed PECL need re-build.
* Cpanel::CachedCommand is wrong.
* Cpanel::OS distro setting is wrong.
* MySQL might now not be upgradable (MySQL versions < 8.0 are not normally present on A8)

## Our current approach can be summarized as:

1. [Check for blockers](Known-blockers)
2. `yum update && reboot`
3. Analyze and remove software (not data) commonly installed on a cPanel system
4. [Execute AlmaLinux upgrade](https://wiki.almalinux.org/elevate/ELevate-quickstart-guide.html)
5. Re-install previoulsy removed software detected prior to upgrade. This might include:
  * cPanel (upcp)
  * EA4
  * MySQL variants
  * Distro Perl/PECL binary re-installs
6. Final reboot (assure all services are running on new binaries)

## RISKS

As always, upgrades can lead to data loss or behavior changes that may leave you with a broken system.

Failure states include but are not limited to:

* Failure to upgrade the kernel due to custom drivers
* Incomplete upgrade of software because this code base is not aware of it.

We recommend you back up (and ideally snapshot) your system so it can be easily restored before continuing.

This upgrade will potentially take 30-90 minutes to upgrade all of the software. During most of this time, the server will be degraded and non-functional. We attempt to disable most of the software so that external systems will re-try later rather than fail in an unexpected way. However there are small windows where the unexpected failures leading to some data loss may occur.

**DISCLAIMER:** We do not guarantee the functionality of software in this repository, and we provide it on an experimental basis only. You assume all risk for any software that you install from this experimental repository. Installation of this software could cause significant functionality failures, even for experienced administrators.

Good Luck!

* [Report bugs here](https://github.com/cpanel/elevate/issues)
* [Code contribtions](https://github.com/cpanel/elevate/pulls) are also welcome!
