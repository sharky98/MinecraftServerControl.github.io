---
layout: default
title: Installation
nav_order: 2
permalink: /docs/mscs/installation
---

# Installation
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Dependencies

We've made an attempt to utilize only features that are normally installed in most Linux and UNIX environments in this script. However, there may be a few requirements that this script has that may not already be in place:
  
- Java JRE - The Minecraft server software requires this. **As of Minecraft version 1.20.5, Java 21 is required as the minimum java version.**<br>
- Perl - Most, if not all, Unix and Linux like systems have this preinstalled.<br>
- libjson-perl - Allows the script to read JSON formatted data.<br>
- libwww-perl - Allows the script to download data to verify downloads.<br>
- liblwp-protocol-https-perl - Allows the script to download data over HTTPS.<br>
- util-linux - Allows the script to use the `flock` script which ships with it for crash detection. Standard package with linux.<br>
- Python - Required by the Minecraft Overviewer mapping software.<br>
- GNU Make - Allows you to use the Makefile to simplify installation.<br>
- GNU Wget - Allows the script to download software updates via the internet.<br>
- rdiff-backup - Allows the script to efficiently run backups.<br>
- rsync - Allows the script to efficiently make copies of files.<br>
- Socat - Allows the script to communicate with the Minecraft server.<br>
- Iptables - Although not explicitly required, a good firewall should be installed.<br>
- Sudo - Run processes under other user and groups  <br>

### Debian or Ubuntu
If you are running Debian or Ubuntu, you can make sure that the dependencies are installed by running the following command:

```bash
sudo apt-get install default-jre perl libjson-perl libwww-perl liblwp-protocol-https-perl util-linux python make wget git rdiff-backup rsync socat iptables
```

**Note**: the version of Java that is shipped in the `default-jre` package, which is the official Debian / Ubuntu Java package, varies based on which version of Debian or Ubuntu you have installed on your system. In some cases (depending on what OS you're running), the version of Java that is shipped with the `default-jre` package is less than Java 21, which is required for Minecraft 1.20.5+. You can test to see if the version of Debian or Ubuntu you have has an official Java 21 package repo by trying: `sudo apt-get install openjdk-21-jre`. If this fails, and you want to play Minecraft versions 1.20.5+, you will either have to download Java 21 manually or add it from an unofficial, third party package repository. Java is backwards compatible, that means that you can use Java 21 for Minecraft 1.8 even if Mojang doesn't state that it is compatible with Java 21

### Fedora, Redhat, or CentOS
If you are running Fedora, Redhat, or CentOS, you can make sure that the dependencies are installed by running the following command:

```bash
sudo dnf install java-21-openjdk perl perl-JSON perl-libwww-perl perl-LWP-Protocol-https util-linux python3 make wget git rdiff-backup rsync socat iptables sudo procps which
```
**Note:** Minecraft 1.18-1.20.4 requires Java 17 or higher. Minecraft 1.20.5+ requires Java 21 or higher

**Additional Note:** if you are running Redhat Enterprise Linux, you must install and enable the EPEL repository in order to install some of the required packages. You can do so with the following commands (assuming you are running RHEL 8):
```bash
sudo subscription-manager repos --enable codeready-builder-for-rhel-8-$(arch)-rpms
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

---

## Configuring the Firewall / NAT
If you are going to run the Minecraft server on your computer, you may need to route some ports to the server (if you
are running the Minecraft server with a hosting company, they most likely already have the ports open). Instructions on
how to accomplish this are beyond the scope of this document, but here are some things you will need to know:

- The default port for the Minecraft server is: 25565.
- If you wish to run multiple world servers using this script, you may want to open a range of ports
  (for example 25565 - 25575).
- If you are using BungeeCord, you will most likely need to only open the default port: 25565.

See the [iptables.rules][iptables_rules] file for a very basic set of rules that you can use with the Iptables firewall.

---

## Quick Start

The fastest way to install the script is to clone the git repository and run the included Makefile:

```bash
git clone https://github.com/MinecraftServerControl/mscs.git && cd mscs
sudo make install
```

This will, by default, create a user to perform MSCS tasks called `minecraft` and give it access to write in the
`/opt/mscs` folder. If there are no errors, then you can proceed to use the script.

[Getting started](getting-started){: .btn .btn-purple }

If you get a `permission denied` error, please see the [troubleshooting](troubleshooting-issues) page.

---

## Manual Installation

If the instructions above do not work (i.e. fails on `sudo make install`) or if the installation requires custom
locations, user accounts, or settings, then follow the instructions below.

To get a server to run the MSCS script on startup, and cleanly stop the server on shutdown,
the [MSCS][mscs] script must be copied to `/usr/local/bin/`, have execute permissions set, and the system must run the
script on startup and shutdown. For Bash completion support, the `mscs.completion` script must be copied to
`/etc/bash_completion.d/`. For security reasons, the script runs with an account named `minecraft` rather than `root`.
The `minecraft` account must be created before the script is used.

1. Create the `minecraft` user:

```bash
useradd --system --user-group --create-home -K UMASK=0022 --home /opt/mscs minecraft
```

2. Install the script:

```bash
sudo install -m 0755 msctl /usr/local/bin/msctl
sudo install -m 0755 mscs /usr/local/bin/mscs
```

3. If using systemd (ie. Ubuntu 15.04+), link the script to the server's startup and shutdown sequences:

```bash
sudo install -m 0644 mscs.service /etc/systemd/system/mscs.service
sudo systemctl -f enable mscs.service
```

4. If using SysV-style, upstart, or similar (ie. Ubuntu 14.10 and lower) link the script to your server's startup and
shutdown sequences:

```bash
sudo ln -s /usr/local/bin/mscs /etc/init.d/mscs
sudo update-rc.d mscs defaults
```

5. Add Bash Completion support:

```bash
sudo install -m 0644 mscs.completion /etc/bash_completion.d/mscs
```

The Minecraft server software will be automatically downloaded to the following location on the first run:

```bash
/opt/mscs/server/
```

---

## Multi-user Setup

MSCS has the ability to store server and world data on a user-by-user basis, allowing multiple users to run their
respective worlds while preserving the data of other user worlds.

> Example: Two user accounts are available on a server: `bob` and `jason`.  
> `bob` should only be able to run and modify Bob's worlds.  
> `jason` should only be able to run and modify Jason's worlds.  
> Each user can have their own properties and world locations

To accomplish this, direct users to use `msctl` in place of `mscs` when running commands
(see [command reference](command-reference)). That's it!

For instance, to create a world named `world`:

```bash
msctl create world 25565
```

To start `world`:

```bash
msctl start world
```

For all commands, replace the `mscs` prefix with `msctl`.

By default, the location of each users' worlds will be saved to `$HOME/mscs/worlds`, where `$HOME` is the home directory
of the user. So if `bob` is logged in, and Bob's home directory is `/home/bob`, Bob's worlds will be saved to
`/home/bob/mscs/worlds`.

**Please note: MSCS currently does not check if a server port is free for use when creating or running worlds. All users
will need to coordinate which ports are being used so no conflicts occur.**

[iptables_rules]: https://github.com/MinecraftServerControl/mscs/blob/master/iptables.rules
[mscs]: https://github.com/MinecraftServerControl/mscs/blob/master/mscs
