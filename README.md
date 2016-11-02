# oro-platform-vagrant #

Vagrant provision script for OroPlatform (and OroCRM, OroCommerce) quick and simple environment setup.

* [About](#About)
  * [What is this project for (and not for)](#What is this project for (and not for))
  * [What inside](#What inside)
* [Installation and configuration](#Installation and configuration)
  * [Requirements](#Requirements)
  * [Installation steps](#Installation steps)
  * [Caveats to installation process](#Caveats to installation process)
  * [Customize installation process](#Customize installation process)
* [Everyday use](#Everyday use)
  * [Shared work folder](#Shared work folder)
  * [SSH access to virtual machine](#SSH access to virtual machine)
  * [Access to database in virtual machine](#Access to database in virtual machine)
  * [Main Vagrant commands](#Main Vagrant commands)

## About ##

### What is this project for (and not for) ###

This project purposes:

* Provide "Quick-start" kit for tasting of OroPlatform-based applications
* Provide simple development environment setup process for OroPlatform-based applications

So, main requirement this project tries to satisfy: quick, smooth, minimum steps installation process.

This is made possible by

* leverage the Vagrant possibilities for quick virtual environments setup
* make reasonable assumptions about all essential installation settings

_If you not satisfied with installation settings defaults, you can change them before (or after) 
installation (please see [Customize installation process](#Customize installation process) 
section below for details). And, of course, you can fork this repository and change files as you want._

**Warning!** To setup **production** environment for OroPlatform-based applications you'd better follow 
official [System requirements][1] and [Installation and Configuration][2] guides.

### What inside ###

The environment based on Ubuntu 16.04 official Vagrant basebox ("ubuntu/xenial64").

In top of this box provision script performs installation steps based on OroCRM
[Installation and Configuratoin][2] guide.

There is following software inside:

* LEMP stack
  * Ubuntu 16.04 LTS
  * Nginx 1.10.*
  * MySQL 5.7.*
  * PHP 7.0.*
* Additions
  * git 2.7.*
  * composer 1.2.*
  * nodejs 4.2.*

## Installation and configuration ##

### Requirements ###

* Windows / MacOS / Linux (or other OS supported by Virtualbox and Vagrant).
* 3072M available RAM on your PC.

_You can decrease reserved RAM to 2048M (or even 1024M) (by edit "vb.memory' param in Vagrantfile),
but it may be not enough for installation process  - depends on your CPU frequency, chosen version of 
Oro application, etc._

### Installation steps ###

First of all, you need to

* Install [Virtualbox][3]
* Install [Vagrant][4]

When you have Virtualbox and Vagrant installed, next steps are simple as one-two-three:

1. Make work directory
2. Put [Vagrantfile][5] from this repository in the work directory (git clone or just download)
3. Run "vagrant up" command in terminal in work directory

Simplest command sequence for OroPlatform application (assumes you have Virtualbox, Vagrant and Git already installed):
```shell
$ git clone https://github.com/NiMias/oro-platform-vagrant.git oroplatform
$ cd oroplatform
$ vagrant up
```

_If you want to install other that OroPlatform application (OroCRM, OroCommerce or your own OroPlatform-based application)
you have to modify parameter GIT_REPO (and, maybe, GIT_TAG) in "Provision configuration" section of
the [Vagrantfile][5]. For more details, please see 
[Customize installation process](#Customize installation process) section below._

### Caveats to installation process ###

First-time installation on your PC may take some time, for

* Downloading Ununtu 16.04 base box
* Installation LEMP stack on guest system
* Installation composer dependencies for Oro application
* Oro application installation process (with, probably, demo data loading).

The time depends on your internet connection speed, CPU frequency, etc. In average it's 15-30 mins.

During installation process might be the pauses. It's ok, because not all installation steps have detailed
command-line output. Please, just wait for several minutes.

After installation finished, you can access application frontend by url http://localhost:8000/ in your browser 
with admin credentials:

* login: admin
* password: adminpass

_You can change application host, admin login and password by modifying installation settings in 
[Vagrantfile][5] before run `vagrant up` command. Please, see "Customize installation process ->
[Oro application settings](#Oro application settings)" section below for details._

At the first time visit to the site server response may be slow because of lack of application cache. So, please,
be patient. If page doesn't respond after 5 min timeout - try to reload it.

### Customize installation process ###

For simplicity there are some default values of installation settings, that can be changed by you before installation
process run if you need it.

These settings situated in "Provision configuration" section of [Vagrantfile][5].

Below described some of them with their default values:

#### Database settings ####

    DB_USER=dbuser
    DB_PASSWORD=dbpass
    DB_NAME=oro

#### Oro application settings ####

    APP_HOST=localhost
    APP_USER=admin
    APP_PASSWORD=adminpass
    APP_LOAD_DEMO_DATA=y    # y | n (whether to perform loading demo data during installation)

**Warning!** If you want to change local application hostname from `localhost` to some other hostname (for example,
`yourdesireddomain.local`), besides modifying APP_HOST variable value you also have to add following record
 in your system `hosts` file:
```shell
192.168.33.10 yourdesireddomain.local www.yourdesireddomain.local
```
Than you can access your installed Oro application in browser by url http://yourdesireddomain.local/.

_For more information about `hosts` file, please, read the [Wikipedia article][6]._

#### Git settings ####

    GIT_REPO="https://github.com/orocrm/crm-application.git"
    GIT_TAG="1.10.8"
    
**Warning!** Git settings have special meaning:

* If variable GIT_REPO **defined** - installation script will attempt to download source files from given 
GIT_REPO url.
* If variable GIT_REPO **NOT defined** (commented or deleted from [Vagrantfile][5]) - it assumes, that you 
have placed application source files in work (current) folder by yourself before running "vagrant up".

_For more information about installation settings, please, see source code and comments in "Provision configuration" section 
of the [Vagrantfile][5]._

## Everyday use ##

### Shared work folder ###

The work folder on your host machine will be visible as `/home/ubuntu/www` folder inside virtual machine.

Hence, any changes you provide to files in work folder on your host machine would be immediately available
in `/home/ubuntu/www` folder inside your virtual machine and vice versa.

### SSH access to virtual machine ###

To get SSH access to virtual machine just type `vagrant ssh` in command line in your work folder on host
machine (without any additional credentials).

Inside virtual machine you have `sudo` permission without password entering.

In case you need SSH access to virtual machine from some utils (like IDE) on your host machine, you can run 
command `vagrant ssh-config` in terminal in work folder and you'll see the SSH credentials, configured by 
Vagrant during installation process.

### Access to database in virtual machine ###

#### Inside virtual machine ####

Inside virtual machine you have access to mysql with credentials provided in "Provision configuration" section
of the [Vagrantfile][5]. By default are:

* `root:dbpass` for root user
* `dbuser:dbpass` for Oro application user

#### From host machine ####

To get access to database on virtual machine from your host machine you need to configure access over SSH
using SSH credentials, that you can find out as described in the 
[SSH access to virtual machine](#SSH access to virtual machine) section above.
    
### Main Vagrant commands ###

#### `vagrant up`

Runs the virtual machine.

At the first time also runs provision script (defined in "config.vm.provision" variable
of [Vagrantfile][5]).

At the second and next times just "wakes up" virtual machine instance without provision script running.

#### `vagrant halt`

Stops the virtual machine.

Save the virtual machine image (without current RAM state) in hard drive of the host machine.

#### `vagrant suspend`

Stops the virtual machine.

Save entire current virtual machine image (with RAM state) ih hard drive of the host machine.

#### Others #####
More detailed information you can find in official [Vagrant documentation][7].

[1]: https://www.orocrm.com/documentation/index/current/system-requirements
[2]: https://www.orocrm.com/documentation/index/current/book/installation
[3]: https://www.virtualbox.org/wiki/Downloads
[4]: https://www.vagrantup.com/docs/installation/
[5]: Vagrantfile
[6]: https://en.wikipedia.org/wiki/Hosts_(file)
[7]: https://www.vagrantup.com/docs/