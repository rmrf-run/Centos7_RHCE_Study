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
  config.vm.synced_folder ".", "/vagrant", disabled: 'true'
  config.vm.synced_folder ".", "/vagrant/rhce", create: 'true'
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
			vb.customize ['createhd', 
					'--filename', 
					'server1-drive1.vhd', 
					'--size', 1024]
      		end
      vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', 'server1-drive1.vhd']
        	if ARGV[0] == "up"
			vb.customize ['createhd', 
					'--filename', 
					'server1-drive2.vhd', 
					'--size', 1024]
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
			vb.customize ['createhd', 
					'--filename', 
					'server2-drive1.vhd', 
					'--size', 1024]
      		end
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', 'server2-drive1.vhd']
      		if ARGV[0] == "up"
			vb.customize ['createhd', 
					'--filename', 
					'server2-drive2.vhd', 
					'--size', 1024]
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
yum -y --disableplugin=fastestmirror install epel-release xorg-x11-xauth mc vim expect setroubleshoot setroubleshoot-server setroubleshoot-plugins selinux-policy selinux-policy-devel policycoreutils-python
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
yum -y --disableplugin=fastestmirror install ipa-server bind-dyndb-ldap
systemctl isolate multi-user.target
#check in ipa folder for setup commands
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-service=ldap
firewall-cmd --permanent --add-service=ldaps
firewall-cmd --permanent --add-service=kerberos
firewall-cmd --permanent --add-port=88/udp
firewall-cmd --permanent --add-port=464/udp
firewall-cmd --permanent --add-port=123/udp
firewall-cmd --reload
ipa-server-install --realm=RHCE.LAB --domain=rhce.lab --ds-password=password --master-password=password --admin-password=password --mkhomedir --hostname=ipa.rhce.lab --ip-address=192.168.123.230 -U
echo "password" | kinit admin
klist
echo "password" | ipa user-add lisa --first=lisa --last=jones --password
echo "password" | ipa user-add linda --first=linda --last=thomsen --password
ipa host-add --force server1.rhce.lab
ipa host-add --force server2.rhce.lab
ipa service-add --force nfs/server1.rhce.lab
ipa service-add --force nfs/server2.rhce.lab
ipa service-add --force cifs/server1.rhce.lab
ipa service-add --force cifs/server2.rhce.lab
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
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
useradd -s /sbin/nologin suser0
useradd -s /sbin/nologin suser1
useradd -s /sbin/nologin suser2
useradd -s /sbin/nologin tuser0
useradd -s /sbin/nologin tuser1
useradd -s /sbin/nologin tuser2
yum -y --disableplugin=fastestmirror install ipa-client setroubleshoot setroubleshoot-server setroubleshoot-plugins selinux-policy selinux-policy-devel policycoreutils-python
semodule -i /vagrant/rhce/selinux/vmblock.pp
semodule -R
systemctl restart httpd
systemctl enable httpd
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
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
useradd -s /sbin/nologin suser0
useradd -s /sbin/nologin suser1
useradd -s /sbin/nologin suser2
useradd -s /sbin/nologin tuser0
useradd -s /sbin/nologin tuser1
useradd -s /sbin/nologin tuser2
yum -y --disableplugin=fastestmirror install ipa-client setroubleshoot setroubleshoot-server setroubleshoot-plugins selinux-policy selinux-policy-devel policycoreutils-python
#ipa-getkeytab -s ipa.rhce.lab -p nfs/server2.rhce.lab -k /etc/krb5.keytab
SCRIPT
end
