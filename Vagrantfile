# -*- mode: ruby -*-
# vi: set ft=ruby :
#
Vagrant.configure(2) do |config|
  # 
  # Vagrant boxes for libvirt or virtualbox
  # 
  #Change to bento/centos-7.2 if you didn't build the box with packer
  #  packer build packer.json
  config.vm.box = "centos7.2"
  #comment below line if you didn't build with packer
  config.vm.box_url = "./centos7.2.box"
  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1024"]
	if ARGV[0] == "up"	
	    vb.customize ["storagectl", :id, "--add", "sata", "--name", "SATA Controller" , "--portcount", 2, "--hostiocache", "on"]
  	end
  end
  config.ssh.forward_x11  = true
#
# 
#
  config.vm.define :server1 do |server1|
    server1.ssh.forward_x11  = true
    server1.vm.network "private_network", ip: "192.168.123.210"
    server1.vm.network "private_network", ip: "192.168.123.211",virtualbox__intnet: "true",nic_type: "virtio"
    server1.vm.network "private_network", ip: "192.168.123.212",virtualbox__intnet: "true",nic_type: "virtio"
    server1.vm.network "private_network", ip: "192.168.123.213",virtualbox__intnet: "true",nic_type: "virtio"
    server1.vm.hostname = "server1.rhce.lab"
    server1.vm.provision "shell", inline: $server1
    server1.vm.synced_folder "rhce/", "/var/www/html/rhce", 
			owner: "apache", 
			group: "apache", 
			create: "true"
    server1.vm.synced_folder "conf.d/", "/etc/httpd/conf.d"
    server1.vm.provider "virtualbox" do |vb|
		if ARGV[0] == "up"
			vb.customize ['createhd', '--filename', 'server1-drive1.vhd', '--size', 1024]
      		end
      vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', 'server1-drive1.vhd']
        	if ARGV[0] == "up"
			vb.customize ['createhd', '--filename', 'server1-drive2.vhd', '--size', 1024]
      		end
      vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', 'server1-drive2.vhd']
    end
  end
  config.vm.define :server2 do |server2|
    server2.ssh.forward_x11  = true
    server2.vm.network "private_network", ip: "192.168.123.220"
    server2.vm.network "private_network", ip: "192.168.123.221",virtualbox__intnet: "true",nic_type: "virtio"
    server2.vm.network "private_network", ip: "192.168.123.222",virtualbox__intnet: "true",nic_type: "virtio"
    server2.vm.network "private_network", ip: "192.168.123.223",virtualbox__intnet: "true",nic_type: "virtio"
    server2.vm.hostname = "server2.rhce.lab"
    server2.vm.provision "shell", inline: $server2
    server2.vm.provider "virtualbox" do |vb|
      		if ARGV[0] == "up"
			vb.customize ['createhd', '--filename', 'server2-drive1.vhd', '--size', 1024]
      		end
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', 'server2-drive1.vhd']
      		if ARGV[0] == "up"
			vb.customize ['createhd', '--filename', 'server2-drive2.vhd', '--size', 1024]
      		end
      vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', 'server2-drive2.vhd']
    end
  end
  config.vm.define :ipa do |ipa|
    ipa.vm.network "private_network", ip: "192.168.123.230"
    ipa.vm.hostname = "ipa.rhce.lab"
    ipa.vm.synced_folder "./ipa", "/vagrant/ipa"
    ipa.vm.provision "shell", inline: $ipa
  end
#
# Common node provisioning
#
$ipa = <<SCRIPT
#!/bin/sh
>&2 echo Common setup
#
# Check internet connection
#
ping -c 2 -W 2 google-public-dns-a.google.com
if [[ $? != 0 ]]
then
  echo "Can't connect to internet" >&2
  exit 1
fi
#
# Turning off firewalld (clean iptables rules)
#
systemctl stop firewalld.service
systemctl disable firewalld.service
#
# Turn on our interfaces, wich may not be started by vagrant
#
systemctl restart NetworkManager.service
systemctl stop network.service
systemctl start network.service
chkconfig network on
yum -y --disableplugin=fastestmirror install epel-release xorg-x11-xauth mc vim expect
SCRIPT
#
# ipa node provisioning
#
$ipa = <<SCRIPT
#!/bin/sh
>&2 echo Ipa setup
#
echo > /etc/hosts
sed -i 's/ipa.rhce.lab//' /etc/hosts
sed -i 's/ipa//' /etc/hosts
echo -e "192.168.123.230 ipa.rhce.lab ipa\n" >> /etc/hosts
echo -e "192.168.123.220 server2.rhce.lab server2\n" >> /etc/hosts
echo -e "192.168.123.210 server1.rhce.lab server1\n" >> /etc/hosts
systemctl restart NetworkManager
yum -y --disableplugin=fastestmirror update
yum -y --disableplugin=fastestmirror install ipa-server bind-dyndb-ldap expect setroubleshoot* selinux-policy policycoreutils-python
systemctl isolate multi-user.target
#check in ipa folder for setup commands
SCRIPT
#
# server1 node provisioning
#
$server1 = <<SCRIPT
#!/bin/sh
>&2 echo Server1 setup
echo > /etc/hosts
sed -i 's/server1.rhce.lab//' /etc/hosts
sed -i 's/server1//' /etc/hosts
echo -e "192.168.123.220 server2.rhce.lab server2\n" >> /etc/hosts
echo -e "192.168.123.210 server1.rhce.lab server1\n" >> /etc/hosts
echo -e "192.168.123.230 ipa.rhce.lab ipa\n" >> /etc/hosts
systemctl restart NetworkManager
systemctl enable firewalld.service
systemctl start firewalld.service
useradd -s /sbin/nologin suser0
useradd -s /sbin/nologin suser1
useradd -s /sbin/nologin suser2
useradd -s /sbin/nologin tuser0
useradd -s /sbin/nologin tuser1
useradd -s /sbin/nologin tuser2
yum -y --disableplugin=fastestmirror install ipa-client setroubleshoot* selinux-policy policycoreutils-python
#ipa-getkeytab -s ipa.rhce.lab -p nfs/server1.rhce.lab -k /etc/krb5.keytab
SCRIPT
#
# server2 node provisioning
#
$server2 = <<SCRIPT
#!/bin/sh
>&2 echo Server2 setup
echo > /etc/hosts
sed -i 's/server2.rhce.lab//' /etc/hosts
sed -i 's/server2//' /etc/hosts
echo -e "192.168.123.220 server2.rhce.lab server2\n" >> /etc/hosts
echo -e "192.168.123.210 server1.rhce.lab server1\n" >> /etc/hosts
echo -e "192.168.123.230 ipa.rhce.lab ipa\n" >> /etc/hosts
systemctl restart NetworkManager
systemctl enable firewalld.service
systemctl start firewalld.service
useradd -s /sbin/nologin suser0
useradd -s /sbin/nologin suser1
useradd -s /sbin/nologin suser2
useradd -s /sbin/nologin tuser0
useradd -s /sbin/nologin tuser1
useradd -s /sbin/nologin tuser2
yum -y --disableplugin=fastestmirror install ipa-client setroubleshoot* selinux-policy policycoreutils-python
#ipa-getkeytab -s ipa.rhce.lab -p nfs/server2.rhce.lab -k /etc/krb5.keytab
SCRIPT
end
