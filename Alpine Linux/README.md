[Go To Overview](../)

# Setup Alpine Linux as docker host in Virtualbox

## Requirement

- Virtalbox

## Download Alpine image

Download an image from [here](https://www.alpinelinux.org/downloads/). For example the *Virtual* version.

## Setup Virtualbox

- Open Virtualbox
- Click on **New**
    - First dialog:
        - Name: **\<choose a name>**
        - Type: **Linux**
        - Version: **Other Lunix (64 or 32, depends on what you downloaded)**
    - Second dialog:
        - 512MB should be the default. You can choose more to be save, it always depends on what you want to do with it.
    - Third dialog:
        - Create a new or use an existing (but empty) storage
- Select the newly created virtual machine in virtualbox
- Click on **Change**
    - Select **Massstorage**
        - Remove the empty disk
        - Click on add disk
            - Select the Alpine ISO
    - Select **Network**
        - Select **Adapter 2**
            - Tick enable
            - choose **Host-only Adapter**
- Click **Start**

## Setup Alpine

execute the following command to apply the default configuration:

````shell
setup-alpine -q
````

> more information can befound in the [alpine wiki: Alpine setup scripts](https://wiki.alpinelinux.org/wiki/Alpine_setup_scripts)

Check if the network was configured correctly with:

````shell
ifconfig
````

Ther should be an **eth0**, **eth1** and **lo**. If it works you can skip the following sub topic.

### Network

Use the following command to edit the network settings:

````shell
vi /etc/network/interfaces
````

Start editing with **:e** and edit the config so that it looks like the folowing:

````config
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet dhcp
        hostname alpine
````

After editing press **[ESC]** and enter **:wq** to save and exit the file.

Aktivate the changes with the following command:

````shell
/etc/init.d/networking restart
````

> [More information about editing the network](https://wiki.alpinelinux.org/wiki/Configure_Networking#Interface_Configuration)

### Setup Repositories

Edit the repositories with the following command:

````shell
vi /etc/apk/repositories
````


Start editing with **:e** and add the following entities:

````config
http://dl-cdn.alpinelinux.org/alpine/v3.8/main
http://dl-cdn.alpinelinux.org/alpine/v3.8/community
`````

Press **[ESC]** and enter **:wq** to save and exit the file.

Now you can add all packages from the following url:
https://pkgs.alpinelinux.org/packages

> [More about package management](https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management)

### Setup Docker and ssh

Use the following command to install docker, ssh and nano

````shell
apk add --no-cache docker, openssh, nano
````

Add docker and ssh to the startup services

````shell
rc-update add docker boot
rc-update add sshd
````

Start both services

````shell
service docker start
/etc/init.d/sshd start
````

> [install docker wiki](https://wiki.alpinelinux.org/wiki/Docker)

> [install ssh-server](https://wiki.alpinelinux.org/wiki/Setting_up_a_ssh-server)


## Create a Snapshot of the VM in Virtualbox

TODO

## Connect to the VM with Putty

TODO