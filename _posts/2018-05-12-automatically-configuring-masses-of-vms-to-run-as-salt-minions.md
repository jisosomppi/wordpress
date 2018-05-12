---
ID: 471
post_title: >
  Automatically configuring masses of VMs
  to run as Salt minions
author: jussi
post_excerpt: ""
layout: post
permalink: >
  http://jisosomppi.me/2018/05/12/automatically-configuring-masses-of-vms-to-run-as-salt-minions/
published: true
post_date: 2018-05-12 10:12:16
---
# manymachines
An attempt to take over the world with numerous salt minions

Main repository available at: [https://github.com/jisosomppi/mgmt](https://github.com/jisosomppi/mgmt)


## Usage
Current method for creating new salt colonies is installing Xubuntu and running:

``` bash
wget jisosomppi.me/deadsea.sh
bash deadsea.sh

```

To reset computers to blank state:
``` bash
cd ~/vagrant/
vagrant destroy -f
vagrant box remove rdyslave
cd
rm -rf ~/vagrant/
rm -rf ~/manymachines/
rm -rf VirtualBox VMs/


```

_Above URL is just an easily memorable version of [https://raw.githubusercontent.com/jisosomppi/manymachines/master/deadsea.sh](https://raw.githubusercontent.com/jisosomppi/manymachines/master/deadsea.sh)

The above script: 
* clones this repository
* installs required programs
* sets up a new base box for vagrant with:
  * updated packages
  * salt-minion installed
  * correct master IP and minion ID set up
* attempts to create 100 VMs per physical machine

## Base box
The base box is created with the script `basebox.sh` using a minimal/trusty64 (Ubuntu 14.04 with minimal clutter) VM as a base, which then gets updated packages so they only need to be fetched once per physical machine, instead of once per VM. The base box gets the packages from the master, which is also configured as an apt-deb-proxy. This saves tons of time in the creation of VMs, since the fetching of packages (even from the proxy) was taking most of the time while bringing up new VMs.

This VM is then packaged into a new base box, which is added to Vagrant so it can be reused.

## Making minions
After the base box is made, the next script (`clonebox.sh`) starts creating new VMs. These new installs already have all the needed packages, as well as the master IP address setup. Each physical computer also gets a random 6 character long alphanumeric string, which is used to identify each physical host. This string is inserted into the VM hostnames to make sure no two Salt minions have the same ID. For some reason though, the minion ID didn't automatically change to the new hostname after VM creation (it retained the `trusty64` hostname of the base box), so I just forced the deletion of the previous ID and confirmed that restarting the salt-minion service creates a new ID file (`/etc/salt/minion_id`). This is the smallest amount of commands in the provision script that I could think of, which will still give me functional salt-minions. This should lead to the fastest possible startup time per VM.

*Note: I thought about creating VMs simultaneously, but that's not a supported feature withing Virtualbox*

## First round of testing

I started my first round of testing by installing Xubuntu 16.04.3 on around 15 computers and running my setup script on all of them.

School computers seem to implode at around 45-55 VMs, despite them having more than enough resources available. For some reason, some of the VMs eat up CPU resources, which in turn causes the host computer to hang and stop creating VMs. Lack of memory on my salt-master is another issue - I had to shut down my WordPress site in order to make enough memory available for even the simplest commands (`sudo salt '*' test.ping --summary` - sadly, the summary counts unresponsive minions as functional despite there being a separate line for them in the output). Lack of memory on the host was also causing interesting errors while attempting to run salt commands.

In the end of round one, I had around 560 minions approved on the master, but a large part of them was unresponsive due to lack of resources on the hosts. 

**Update: Apparently the high CPU load is due to the VMs running out of memory. Moving data to the swap space is CPU intensive and causes the host computer to stall. Attempting to fix by increasing VM RAM slightly.**
