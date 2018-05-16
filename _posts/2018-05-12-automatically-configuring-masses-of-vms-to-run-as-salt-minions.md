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
An attempt to take over the world with numerous salt minions, or

***"How to run 2000+ VMs within your line of sight"***  
[(List of controlled minions here)](https://github.com/jisosomppi/manymachines/blob/master/extras/minionlist2k)

![Final result](https://github.com/jisosomppi/manymachines/blob/master/images/IMG_20180514_163012.jpg?raw=true)

This is my answer to a challenge from [Tero Karvinen](http://terokarvinen.com), issued during a Haaga-Helia course, for which the main repository available at: [https://github.com/jisosomppi/mgmt](https://github.com/jisosomppi/mgmt)


## Usage
Current method for creating new salt colonies is installing Xubuntu and running:

``` bash
wget jisosomppi.me/deadsea.sh
bash deadsea.sh

```

_Above URL is just an easily memorable link for [the script in this repository,](https://raw.githubusercontent.com/jisosomppi/manymachines/master/deadsea.sh) all changes are made to files in this repository._

_The IP address for the salt master is defined in Vagrantfile_basebox_

_The target amount of VMs per host is defined Vagrantfile_rdyslave_


To reset computers to blank state:
``` bash
cd ~/vagrant/
vagrant destroy -f
vagrant box remove rdyslave
cd
rm -rf ~/vagrant/
rm -rf ~/manymachines/
rm -rf VirtualBox\ VMs/


```

The above script: 
* clones this repository
* installs required programs
* sets up a new base box for vagrant with:
  * updated packages
  * salt-minion installed
  * correct master IP and minion ID set up
* attempts to create 100 VMs per physical machine

## Base box
The base box is created with the script `basebox.sh` using a minimal/trusty64 (Ubuntu 14.04 with minimal clutter) VM as a base, which then gets updated packages so they only need to be fetched once per physical machine, instead of once per VM. To shave seconds off the load times, I set up and apt-deb-proxy on my master server as well. This saves tons of time in the creation of VMs, since the fetching of packages (even from the proxy) was taking most of the time while bringing up new VMs.

This VM is then packaged into a new base box, which is added to Vagrant so it can be reused.

## Making minions
After the base box is made, the next script (`clonebox.sh`) starts creating new VMs. These new installs already have all the needed packages, as well as the master IP address setup. Each physical computer also gets a random 6 character long alphanumeric string, which is used to identify each physical host. This string is inserted into the VM hostnames to make sure no two Salt minions have the same ID. For some reason though, the minion ID didn't automatically change to the new hostname after VM creation (it retained the `trusty64` hostname of the base box, even after restarting salt-minion), so I just forced the deletion of the previous ID and confirmed that restarting the salt-minion service creates a new ID file (`/etc/salt/minion_id`). This is the smallest amount of commands in the provision script that I could think of, which will still give me functional salt-minions. This should lead to the fastest possible startup time per VM.

## First round of testing

I started the first round of testing by installing Xubuntu 16.04.3 on around 15 computers and running my setup script on all of them.

School computers seem to implode at around 45-55 VMs, despite them having more than enough resources available. For some reason, some of the VMs eat up CPU resources, which in turn causes the host computer to hang and stop creating VMs. Lack of memory on my salt-master is another issue - I was running the salt master on my server, and had to shut down my WordPress site in order to make enough memory available for even the simplest commands (`sudo salt '*' test.ping --summary` - sadly, the summary counts unresponsive minions as functional despite there being a separate line for them in the output). Lack of memory on the host was also causing interesting errors while attempting to run salt commands.

In the end of round one, I had around 560 minions approved on the master, but a large part of them was unresponsive due to lack of resources on the hosts. 

**Update: Apparently the high CPU load is due to the VMs running out of memory. Moving data to the swap space is CPU intensive and causes the host computer to stall. Attempting to fix by increasing VM RAM slightly.**

## Second and final round of testing 

![Creating 25 * 80 Vagrant VMs](https://github.com/jisosomppi/manymachines/blob/master/images/IMG_20180514_154231.jpg?raw=true)

### Process
After allocating slightly more RAM per VM, I managed to get 80 VMs running on (nearly) each physical host. Running the script wasn't a 100% solid task, and I had to manually restart the `vagrant up` process on around 5 of the 25 machines in room 5004. In most of the cases this was caused by a failure in the Vagrant host <-> VM SSH connection, an error that didn't reoccur during another `vagrant up` run. 

In one case, the process was stopped due to the requested port (50050) being in use on the host computer. I didn't find out what process was using it, but a new run of `vagrant up` passed without errors.

The surprising part was that the main cause of bad results or failed commands wasn't actually the minions, but the master! I was running my master on the same server machine as the 120+ minions, and issuing a command to all of them at the same time spiked the CPU use across all 8 cores to 100%. During this time, it seemed that some minions had tried to return their results to the master, but the master wasn't able to log the responses. This in turn meant that the master decided that those VMs hadn't answered to the commands.

### Tricks
To simplify tracking, it's a good idea to use the --summary modifier on salt commands, as it returns the number of targeted/responding minions. *Extra usefulness: add **time** to the beginning of the commands*

My salt-master started to timeout with 600+ minions, so I needed to improve its performance. This is done easily, by increasing the `worker_threads` value in `/etc/salt/master`, in my case I incrementally raised the value from the default 5 all the way up to 20. *Rule of thumb: 1 thread per 100 minions?*

To reduce the load on both the master and the minions, using the -b or --batch-size modifier should limit simultaneous commands to a specified amount. Although for me, using this modifier caused the command to return nothing at all.

### Results
![Resource usage with 80 VMs running](https://github.com/jisosomppi/manymachines/blob/master/images/IMG_20180514_160759.jpg?raw=true)
After running the install script on 25 machines (plus 120-something VMs on a server machine), I finally reached my target and hit 2000 VMs. This means that each host was running approximately 78 VMs at a time. With 80 VMs running, the memory use of the physical hosts was around 14,5 / 15,5 Gb, leaving some breathing room to ensure host stability.

***The final result was 2071 VMs targeted, out of which 2021 answered to a `test.ping` at the same time.*** 

## Further improvement

This is by no means a perfect execution, and has many things that could be improved in the future. The ones that appear most important to me are:
* Using containers or other similar methods to further reduce resource draw
* Making the amount of VMs scale based on the host computer, with some sort of logic for making sure the machine remains functional
* Setting the host computers up for centralized management as well, in this case I realized this after using the same hostname for all 25 host computers (username+computer model, the default hostname) and didn't have the time to fix my mistake
