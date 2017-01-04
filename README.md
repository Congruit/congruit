![alt text](https://github.com/lucky-sideburn/congruit/blob/master/img/logo2.png "Tux2")
### The configuration management tool that loves Bash
#### Simple, lightweight and fully customizable by you

### Table of Contents

**[Description](#description)**

**[New! Make clusters with congruit)(#ha)**

**[Concepts](#concepts)**

**[Stockroom](#stockroom)**

**[Place](#place)**

**[Work](#work)**

**[Workplace](#workplace)**

**[Build your workplaces](#build-your-workplaces)**

**[Prerequisites](#prerequisites)**

**[Try Congruit With Vagrant](#try-congruit-with-vagrant)**

**[Usage](#usage)**

## Description
Congruit is a lightweight configuration management and automation tool. It is written in Go but works through Bash. It manages shell scripts you created to configure your Linux platforms.

## ha
Congruit manages cluster through supervisor and friend mode. You can configure floating ip or manage Docker clusters.

Let's show ho to put in place a simple docker cluster

1. Start the server Docker01 and Docker02

Leave WORKPLACES_ENABLED empty!

```
vagrant up Docker01
vagrant up Docker02

vagrant provision Docker01
vagrant provision Docker02

vagrant status
Docker01                  running (virtualbox)
Docker02                  running (virtualbox)
```
If all things have done correctly, Congruit will start and wait for commands sent by the cluster controller.

```
==> Docker02: Running provisioner: shell...
    Docker02: Running: inline script
==> Docker02: Running provisioner: shell...
    Docker02: Running: inline script
==> Docker02: Running provisioner: shell...
    Docker02: Running: inline script
==> Docker02:                          _ _
==> Docker02:  ___ ___ ___ ___ ___ _ _|_| |_
==> Docker02: |  _| . |   | . |  _| | | |  _|
==> Docker02: |___|___|_|_|_  |_| |___|_|_|
==> Docker02:             |___|
==> Docker02: Version: 1.0.0
==> Docker02: 2017/01/04 17:11:06 There are no workplaces to apply... Doing nothing...
==> Docker02: 2017/01/04 17:11:06 Extecuted works: 0
```

2. Start dockers with the following command from you workstation:

```
./congruit --stockroom-dir=stockroom-docker-clu-controller/ -supervisor  -debug
````
Note that --stockroom-dir=stockroom-docker-clu-controller/ is not the commond stockroom. It is a stockroom created for run a cluster controller.

Parameters:
* -supervisor => Run congruit continusly
* -friend => starts congruit in friends mode... In this mode a Congruit instance can receive remote command from a controller
* -token => authentication tocket

For keeping up Dockers you can use works like this:

```
curl https://192.168.50.4:8443/hello  --header "Token:foobar" --header "Workplace:tomcat-docker"
curl https://192.168.50.5:8443/hello  --header "Token:foobar" --header "Workplace:tomcat-docker"
```

## Concepts
The main concepts of Congruit are

* Stockroom repository
* Works
* Places
* Workplaces

## Stockroom
The Stockroom is the main repository that describes your platform. Congruit reads the stockroom and does things.

## Place
A place is a shell script that must return 0. You should be in a right place to do a work.
Example:

Is this Linux server a Centos 7?

```
[ ! -e /etc/redhat-release ] && exit 1
cat /etc/redhat-release | grep "Centos Linux release 7.*"
```

## Work
Work is a shell script that installs and configures programs or runs Docker containers like in the following example:

```
docker run --rm -p 8888:8080 tomcat:latest &> /dev/null &
```

## Workplace
Workplaces are the union between works and places and are JSON file.

Example:

```
[
  {
   "places": ["debian","screen_is_not_installed"],
   "works": ["screen_package_apt"]
  },
  {
   "places": ["centos7","screen_is_not_installed"],
   "works": ["screen_package_yum"]
  }
]
```
the workplace is able to decide which is the correct strategy to install software.
Congruit executes places and, if they return 0, it does works.

## Build you workplace
1. Create your stockroom. I would like create a public repository with common and useful workplaces. For now you can take a look at https://github.com/lucky-sideburn/congruit/tree/master/stockroom

2. You need to describe your places. Example:
  * is this server running a specific Linux distribution?
  * are there particular configuration files, installed software, environment variables that describe the role or the functionality of this server?
  * places are executed before works... You can copy files which contain environment variables that can be used by works.
  Put places in stockroom/places/ folder

3. Create works. Put the scripts in stockroom/works. Works install software, get configuration file from a repository, manage Docker containers ecc..

4. Create workplaces in stockroom/workplaces. Try to make them usable in more environments and follow this example:

Workplaces are array of hashes
```
[
  {
   "places": ["is_linux", "is_frontend","is_apache_not_istalled"],
   "works": ["install_apache"]
  },

  {
   "places": ["is_linux", "is_frontend", "has_additionl_vhost"],
   "works": ["additional_vhost","restart_apache"]
  },

  {
   "places": ["is_production"],
   "works": ["do_backup"]
  }

]
```

## Try Congruit With Vagrant

* List all virtual machine in current Vagrant project

```
eugenio@local:[~/WORK/GO/src/congruit]: vagrant status
Current machine states:

Centos7                   running (virtualbox)
```

* Provision and test your workplaces

```
export WORKPLACES_ENABLED=install_screen
vagrant provision Centos7
```

Remember to set WORKPLACES_ENABLED (example: WORKPLACES_ENABLED=do_this,do_this2,do_foobar) in order to execute workplaces

Example of Vagrant's output:

```
==> Centos7:                          _ _
==> Centos7:  ___ ___ ___ ___ ___ _ _|_| |_
==> Centos7: |  _| . |   | . |  _| | | |  _|
==> Centos7: |___|___|_|_|_  |_| |___|_|_|
==> Centos7:             |___|
==> Centos7: Version: 1.0.0
==> Centos7: 2017/01/03 09:02:02 Loading places...
==> Centos7: 2017/01/03 09:02:02 Found place: centos7
==> Centos7: 2017/01/03 09:02:02 Found place: debian
==> Centos7: 2017/01/03 09:02:02 Found place: docker_tomcat_is_not_running
==> Centos7: 2017/01/03 09:02:02 Found place: everywhere
==> Centos7: 2017/01/03 09:02:02 Found place: fedora
==> Centos7: 2017/01/03 09:02:02 Found place: git_is_not_installed
==> Centos7: 2017/01/03 09:02:02 Found place: osx
==> Centos7: 2017/01/03 09:02:02 Found place: screen_is_not_installed
==> Centos7: 2017/01/03 09:02:02 Loading workplaces...
==> Centos7: 2017/01/03 09:02:02 Found workplace: install_screen
==> Centos7: 2017/01/03 09:02:02 Loading workplace: install_screen@1
==> Centos7: 2017/01/03 09:02:02 Loading workplace: install_screen@2
==> Centos7: 2017/01/03 09:02:02 Loading workplace: install_screen@3
==> Centos7: 2017/01/03 09:02:02 Loading works...
==> Centos7: 2017/01/03 09:02:02 Found work: run_tomcat_docker
==> Centos7: 2017/01/03 09:02:02 Found work: screen_package_apt
==> Centos7: 2017/01/03 09:02:02 Found work: screen_package_dnf
==> Centos7: 2017/01/03 09:02:02 Found work: screen_package_yum
==> Centos7: 2017/01/03 09:02:02
==> Centos7:  ***
==> Centos7:  Going to apply workplaces
==> Centos7:  ***
==> Centos7: 2017/01/03 09:02:02 Workplace: install_screen@1
==> Centos7: 2017/01/03 09:02:02 Checking places...
==> Centos7: 2017/01/03 09:02:02 Testing Place: debian
==> Centos7: 2017/01/03 09:02:02 Executing Place:
==> Centos7: [ -e /etc/debian_version ] && exit 0
==> Centos7: 2017/01/03 09:02:02 Workplace install_screen@1 not needed here!
==> Centos7: 2017/01/03 09:02:02 Workplace: install_screen@2
==> Centos7: 2017/01/03 09:02:02 Checking places...
==> Centos7: 2017/01/03 09:02:02 Testing Place: centos7
==> Centos7: 2017/01/03 09:02:02 Executing Place:
==> Centos7: [ ! -e /etc/redhat-release ] && exit 1
==> Centos7: cat /etc/redhat-release | grep -i "Centos Linux release 7.*"
==> Centos7: 2017/01/03 09:02:02 Place execution output: CentOS Linux release 7.2.1511 (Core)
==> Centos7: 2017/01/03 09:02:02 Testing Place: screen_is_not_installed
==> Centos7: 2017/01/03 09:02:02 Executing Place:
==> Centos7: which screen
==> Centos7: if [ $? -ne 0 ]
==> Centos7: then
==> Centos7:   exit 0
==> Centos7: else
==> Centos7:   exit 1
==> Centos7: fi
==> Centos7: 2017/01/03 09:02:02 Place execution output:
==> Centos7: 2017/01/03 09:02:02 Executing Work:
==> Centos7: yum -y install screen
==> Centos7: 2017/01/03 09:02:04 Loaded plugins: fastestmirror
==> Centos7: Loading mirror speeds from cached hostfile
==> Centos7:  * base: mirror.crazynetwork.it
==> Centos7:  * epel: pkg.adfinis-sygroup.ch
==> Centos7:  * extras: mirror.crazynetwork.it
==> Centos7:  * updates: mirror.crazynetwork.it
==> Centos7: Resolving Dependencies
==> Centos7: --> Running transaction check
==> Centos7: ---> Package screen.x86_64 0:4.1.0-0.23.20120314git3c2946.el7_2 will be installed
==> Centos7: --> Finished Dependency Resolution
==> Centos7:
==> Centos7: Dependencies Resolved
==> Centos7:
==> Centos7: ================================================================================
==> Centos7:  Package    Arch       Version                                   Repository
==> Centos7:                                                                            Size
==> Centos7: ================================================================================
==> Centos7: Installing:
==> Centos7:  screen     x86_64     4.1.0-0.23.20120314git3c2946.el7_2        base     552 k
==> Centos7:
==> Centos7: Transaction Summary
==> Centos7: ================================================================================
==> Centos7: Install  1 Package
==> Centos7:
==> Centos7: Total download size: 552 k
==> Centos7: Installed size: 914 k
==> Centos7: Downloading packages:
==> Centos7: Running transaction check
==> Centos7: Running transaction test
==> Centos7: Transaction test succeeded
==> Centos7: Running transaction
==> Centos7:   Installing : screen-4.1.0-0.23.20120314git3c2946.el7_2.x86_64             1/1
==> Centos7:   Verifying  : screen-4.1.0-0.23.20120314git3c2946.el7_2.x86_64             1/1
==> Centos7:
==> Centos7: Installed:
==> Centos7:   screen.x86_64 0:4.1.0-0.23.20120314git3c2946.el7_2
==> Centos7:
==> Centos7: Complete!
==> Centos7: 2017/01/03 09:02:04 Workplace: install_screen@3
==> Centos7: 2017/01/03 09:02:04 Checking places...
==> Centos7: 2017/01/03 09:02:04 Testing Place: fedora
==> Centos7: 2017/01/03 09:02:04 Executing Place:
==> Centos7: [ ! -e /etc/redhat-release ] && exit 1
==> Centos7: grep -i "Fedora release 2[012].*" /etc/redhat-release
==> Centos7: 2017/01/03 09:02:04 Workplace install_screen@3 not needed here!
==> Centos7: 2017/01/03 09:02:04 Extecuted works: 1
``

## Prerequisites
1. GO
2. a stockroom. You can take as example the stockroom present in this repo. Please, create symlink from stockroom/workplaces/foo to stockroom/workplaces_enabled/foo if you want apply the workplace "foo" during congruit execution


## Usage
1. `git clone https://github.com/lucky-sideburn/congruit.git`
2. `go build conguit.go`

3. Start Congruit

`./congruit  -stockroom-dir=./stockroom`

`./congruit  -stockroom-dir=./stockroom -debug`

