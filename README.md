# ynh-dev - Yunohost dev environment manager

Please report issues on the following repository:

> https://github.com/yunohost/issues

# Table Of Contents

- [Introduction](#introduction)
  * [Local Development Path](#local-development-path)
  * [Remote Development Path](#remote-development-path)

- [ynh-dev - Yunohost dev environment manager](#ynh-dev---yunohost-dev-environment-manager)
- [Table Of Contents](#table-of-contents)
- [Introduction](#introduction)
  - [Local Development Path](#local-development-path)
    - [Requirements](#requirements)
      - [On Linux](#on-linux)
      - [On Windows and Mac OS](#on-windows-and-mac-os)
  - [Remote Development Path](#remote-development-path)
- [Local Development Environment](#local-development-environment)
  - [1. Setup `ynh-dev` and the development environment on Linux with LXD](#1-setup-ynh-dev-and-the-development-environment-on-linux-with-lxd)
    - [Installing and configuring LXD](#installing-and-configuring-lxd)
    - [Create a container and install YunoHost in it.](#create-a-container-and-install-yunohost-in-it)
  - [Development and container testing](#development-and-container-testing)
  - [Testing the web interface](#testing-the-web-interface)
  - [2. Manage YunoHost's dev LXCs](#2-manage-yunohosts-dev-lxcs)
  - [Advanced: using snapshots to keep](#advanced-using-snapshots-to-keep)
  - [Alternative: Using Only Virtualbox](#alternative-using-only-virtualbox)
- [Remote Development Environment](#remote-development-environment)
  - [1. Setup your VPS and install YunoHost](#1-setup-your-vps-and-install-yunohost)
  - [2. Setup `ynh-dev` and the development environment](#2-setup-ynh-dev-and-the-development-environment)
  - [3. Develop and test](#3-develop-and-test)
- [Further Resources](#further-resources)

- [Remote Development Environment](#remote-development-environment)
  * [1. Setup your VPS and install YunoHost](#1-setup-your-vps-and-install-yunohost)
  * [2. Setup `ynh-dev` and the development environment](#2-setup-ynh-dev-and-the-development-environment)
  * [3. Develop and test](#3-develop-and-test)

- [Further Resources](#further-resources)

---

# Introduction

`ynh-dev` is a CLI tool to manage your local development environment for YunoHost.

This allow you to develop on the various repositories of the YunoHost project.

In particular, it allows you to:

 * Create a directory with a clone of each repository of the YunoHost project
 * Replace Yunohost debian packages with symlinks to those git clones

Because there are many diverse constraints on the development of the Yunohost
project, there is no "official" one-size-fits-all development environment.
However, we do provide documentation for what developers are using now in
various circumstances.

Please keep this in mind when reviewing the following options with regard to
your capacities and resources when aiming to setup a development environment.

`yhn-dev` can be used for the following scenarios:

## Local Development Path

The local development path allows to work without an internet connection,
but be aware that it will *not* allow you to easily test your email stack
or deal with deploying SSL certificates, for example, as your machine is
likely to not be exposed on the internet. A remote machine should be used
for these cases.

Depending on your needs, this setup can be very convenient.

If choosing this path, please keep reading at the [local development
environment](#local-development-environment) section.

### Requirements

Yunohost can be developed on using a combination of the following technologies:

#### On Linux

The recommanded method used by many YunoHost developpers is to used LXC containers to setup the YunoHost system. LXC is especially more lightweight than Virtualbox or virtualisation. The easiest method to setup LXC container is to used the LXD runtime. So you will need the following:

  * Git (any version is sufficient)
  * Snap to install recent LXD release
  * LXD ( >= 3.x )

LXC is typically lightweight but you may find the initial setup complex
(in particular network configuration).

Alternatively, you may be able to setup a local environnement using Virtualbox which is kinda more
resource-hungry :

  * Virtualbox (>= 5.x)
  * Vagrant-virtualbox

Please see the [Alternative: Only Virtualbox](#alternative-using-only-virtualbox)
section for more.

Please keep in mind that these versions may not be available on your OS
distribution and you may be required to install them as binary or from source.
There are no guarantees of stability on newer major versions.

#### On Windows and Mac OS

LXD is not installable on Windows or Mac OS so you should use the virtualbox method.

## Remote Development Path

Yunohost can be deployed as a typical install on a remote VPS. You can then use
`ynh-dev` to configure a development environment on the server.

This method can potentially be faster than the local development environment
assuming you have familiarity with working on VPS machines, if you always have
internet connectivity when working, and if you're okay with paying a fee. It
is also a good option if the required system dependencies (Vagrant, Virtualbox,
etc.) are not easily available to you on your distribution.

Please be aware that this method should **not** be used for a end-user facing
production environment.

If choosing this path, please keep reading at the [remote development
environment](#remote-development-environment) section.

# Local Development Environment

Here is the development flow:

1. Setup `ynh-dev` and the development environment
2. Manage YunoHost's development LXCs
3. Develop on your local host and testing in the container

## 1. Setup `ynh-dev` and the development environment on Linux with LXD


### Installing and configuring LXD

First you need to install the system dependencies.

`ynh-dev` essentially requires Git and the LXD ecosystem. Please
see the [local development path](#local-development-path) section for some idea
of the versions required.

The following commands should work on **Linux Mint 19** (and possibly on any Debian Stretch?):

```bash
$ git clone https://github.com/yunohost/ynh-dev.git
$ sudo apt update
$ sudo apt install snapd
$ sudo snap install lxd
```

Once LXD is installed we need to initialize it with the following command:

```bash
$ sudo lxd init
```

This interactive command ask you some questions. You can just keep all default options by pushing enter key until the end.

Depending on your setup you should perhaps switch to root user to execute `lxd` and `lxc` commands:

```bash
$ sudo -i
$ lxd init
```

### Create a container and install YunoHost in it.

```bash
$ sudo -i
$ lxc launch images:debian/stretch/amd64 ynh-dev
$ lxc list
```

We can know switch the container to priviledged mode. This is necessary only to have a convienient shared folder between your host environnement and the container. You can avoid the priviledged mode if you don't mind coding directly inside the container.

```bash
$ lxc config set ynh-dev security.priviledged true
$ lxc exec ynh-dev reboot # restart the container
```

Then we can clone ynh-dev and setup the shared folder.

```
$ lxc config device add ynh-dev ynh-dev-shared-folder disk path=/ynh-dev source="./ynh-dev"
```

To go inside the container launch:

```
$ lxc exec ynh-dev /bin/bash
$ ls -al /ynh-dev  # to check the content of the shared folder
```

We can now install yunohost (unstable repo) inside the container:

```
$ apt install curl
$ curl https://install.yunohost.org | bash -s -- -a -d unstable
```

You can now have a cup of coffee since the installation process take some time to finish.
Now that YunoHost is installed you can export an image of the container to recreate it easily next time.
Stop the container from inside:

```
$ poweroff
```

Then export the image with:

```
$ lxc publish ynh-dev "yunohost_unstable_$(date +'%Y%m%d')"
```

Before launching YunoHost post installation you need to get the code source of the part of Yunohost you want to work with and link it into the container using the ynh-dev script.

## Development and container testing

After going inside the container, you should notice that the *directory* `/ynh-dev` is a shared folder with your host. In particular, it contains the various git clones `yunohost`, `yunohost-admin` and so on - as well as the `./ynh-dev` script itself.

Inside the container, `./ynh-dev` can be used to link the git clones living in the host to the code being ran inside the container.

For instance, after running:

```bash
$ ./ynh-dev use-git yunohost
```

The code of the git clone `'yunohost'` will be directly available inside the container. Which mean that running any `yunohost` command inside the container will use the code from the host... This allows to develop with any tool you want on your host, then test the changes in the container.

The `use-git` action can be used for any package among `yunohost`, `yunohost-admin`, `moulinette` and `ssowat` with similar consequences. You might want to run use-git several times depending on what you want to develop precisely.

***Note***: The `use-git` operation can't be reverted now. Do **not** do this in production.

## Testing the web interface

You should be able to access the web interface via the IP address of the container. The IP can be known from inside the container using either from `ip a` or with `./ynh-dev ip`.

If you want to access to the interface using the domain name, you shall tweak your `/etc/hosts` and add a line such as:

```bash
111.222.333.444 yolo.test
```

Note that `./ynh-dev use-git yunohost-admin` has a particular behavior: it starts a `gulp` watcher that shall re-compile automatically any changes in the javascript code. Hence this particular `use-git` will keep running until you kill it after your work is done.

## 2. Manage YunoHost's dev LXCs

When ran on the host, the `./ynh-dev` command allows you to manage YunoHost's dev LXCs.

First, you might want to start a new LXC with:

```bash
$ cd ynh-dev  # if not already done
$ ./ynh-dev start
```

This should download an already built LXC from `build.yunohost.org`. If this does not work (or the LXC is outdated), you might want to (re)build a fresh LXC locally with `./ynh-dev rebuild`.

After starting the LXC, you should be automatically SSH'ed inside. If you later disconnect from the LXC, you can go back in with `./ynh-dev ssh`.

Later, you might want to destroy the LXC. You can do so with `./ynh-dev destroy`.



## Advanced: using snapshots to keep



However, you may still use `lxc-snapshot` directly to manage snapshots.

## Alternative: Using Only Virtualbox

A Vagrant and Virtualbox (without LXC) guide is provided on another branch of
this repository. This is a known working setup used by some developers. Please
see the ["virtualbox" branch](https://github.com/YunoHost/ynh-dev/tree/virtualbox#develop-on-your-local-machine)
for more.



# Remote Development Environment

Here is the development flow:

1. Setup your VPS and install YunoHost
2. Setup `ynh-dev` and the development environment
3. Develop and test

## 1. Setup your VPS and install YunoHost

Setup a VPS somewhere (e.g. Scaleway, Digital Ocean, etc.) and install YunoHost following the [usual instructions](https://yunohost.org/#/install_manually).

Depending on what you want to achieve, you might want to run the postinstall right away - and/or setup a domain with an actually working DNS.

## 2. Setup `ynh-dev` and the development environment

Deploy a `ynh-dev` folder at the root of the filesystem with:

```
$ cd /
$ curl https://raw.githubusercontent.com/yunohost/ynh-dev/master/deploy.sh | bash
$ cd /ynh-dev
```

## 3. Develop and test

Inside the VPS, `./ynh-dev` can be used to link the git clones to actual the code being ran.

For instance, after running:

```bash
$ ./ynh-dev use-git yunohost
```

Any `yunohost` command will run from the code of the git clone.

The `use-git` action can be used for any package among `yunohost`, `yunohost-admin`, `moulinette` and `ssowat` with similar consequences.

# Further Resources

  * [yunohost.org/dev](https://yunohost.org/dev)
