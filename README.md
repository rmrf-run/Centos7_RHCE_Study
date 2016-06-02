# RHCE Lab built with packer and vagrant

_I am not affilated with Red Hat nor do I have any idea what questions are on the RHCE exam. I assembled everything in this repo from the various sources below. Using this lab will not gurantee  you will pass your RHCE exam._
***
### Get Started

* Install Vagrant, Virtualbox, and Packer
* Run ` packer build packer.json ` 
* Run ` vagrant up `
* packer build will take a good while as will vagrant up
* the ` ks.cfg ` file will install most needed packages and run a yum update

### Once installed
* look in selinux folder for custom semodule to allow `httpd_t` access to `vmblock_t`, _this will need to be completed before objectives can be viewed in browser_
* visit http://server1.rhce.lab for objectives
* install any other needed packages to complete objectives
* most objectives will have help links to videos or repos

### VM Details

* server1.rhce.lab is main server, runs httpd for objectives, can be setup as client
* server2.rhce.lab can be setup as client server for NFS, POSTFIX, etc
* ipa.rhce.lab is ipa server checkout the ipa folder for set up details, provides kerberos auth for NFS and User auth as well. I wouldn't do much on this VM
* each box has 4 nics to set up teaming/bonding
* No ssh keys for root user have been copied between machines

### Items still needed

* ipv6 support to configure nic with ipv6

***

### Sources 

* [dreyou/rhce7](https://github.com/dreyou/rhce7)
* [lucklylittle/RHCE](https://github.com/luckylittle/RHCE)
* [k4ch0 / rhce_7](https://github.com/k4ch0/rhce_7)
* [iahmad-khan / RHCE-RHEL7](https://github.com/iahmad-khan/RHCE-RHEL7)
* [Sander van Vugt / Youtube](https://www.youtube.com/channel/UComgXoI6pysmetOzuNH_TDQ)
* [hfm/packer-centos-7](https://github.com/hfm/packer-centos-7)
