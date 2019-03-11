# -*- mode: ruby -*-
# vi: set ft=ruby :

$nnscript = <<-SCRIPT
echo "" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "auto enp0s8" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "iface enp0s8 inet static" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "address 10.11.13.4" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "gateway 10.11.13.1" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "netmask 255.255.255.0" >> /etc/network/interfaces.d/50-cloud-init.cfg
sudo /etc/init.d/networking restart
sudo apt-get update && sudo apt-get install -y nfs-common
sudo mkdir /shared
sudo chmod 777 /shared
sudo echo "10.11.13.3:/export/shared  /shared  nfs  auto  0  0" >> /etc/fstab
sudo mount -a 
sudo apt-get install apache2 -y
SCRIPT

$nn2script = <<-SCRIPT
echo "" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "auto enp0s8" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "iface enp0s8 inet static" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "address 10.11.13.5" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "gateway 10.11.13.1" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "netmask 255.255.255.0" >> /etc/network/interfaces.d/50-cloud-init.cfg
sudo /etc/init.d/networking restart
sudo apt-get update && sudo apt-get install -y nfs-common
sudo mkdir /shared
sudo chmod 777 /shared
sudo echo "10.11.13.3:/export/shared  /shared  nfs  auto  0  0" >> /etc/fstab
sudo mount -a 
sudo apt-get install apache2 -y
SCRIPT

$nmscript = <<-SCRIPT
echo "" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "auto enp0s8" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "iface enp0s8 inet static" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "address 10.11.13.3" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "gateway 10.11.13.1" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "netmask 255.255.255.0" >> /etc/network/interfaces.d/50-cloud-init.cfg
sudo /etc/init.d/networking restart ; true
sudo apt-get update && sudo apt-get -y install nfs-kernel-server
sudo mkdir -p /export/shared
sudo mkdir /shared
sudo chmod 777 /{export,shared} && sudo chmod 777 /export/*
sudo mount --bind /shared /export/shared
echo "/export/shared *(rw,fsid=0,insecure,no_subtree_check,async)" >> /etc/exports
sudo service nfs-kernel-server restart
sudo apt-get -y install haproxy
sudo cp /etc/haproxy/haproxy.cfg /etc/haproxy.real.cfg
echo "
frontend firstbalancer
        bind *:80
        stats uri /haproxy?stats
        option forwardfor
        default_backend webservers

backend webservers
	balance roundrobin
        server www_01 10.11.13.4:80 check
	server www_02 10.11.13.5:80 check
        option httpchk" >> /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
sudo apt-get install docker.io -y
sudo docker pull ashael/openmpi
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.define "master" do |m|
  	m.vm.box = "ubuntu/xenial64"
        m.vm.hostname = "master.localhost"
  	m.vm.provider :virtualbox do |vb|
		vb.customize [ 'modifyvm', :id, '--memory', '1500' ]
		vb.customize [ 'modifyvm', :id, '--cpus', '1' ]
		vb.customize [ 'modifyvm', :id, '--name', 'cluster-master' ]
                vb.customize [ "modifyvm", :id, "--nic2", "hostonly" ]
                vb.customize [ "modifyvm", :id, "--hostonlyadapter2", "vboxnet1" ]
  	end
        m.vm.provision "shell", inline: $nmscript
  end
 
  config.vm.define "node1" do |m|
  	m.vm.box = "ubuntu/xenial64"
        m.vm.hostname = "node.localhost"
  	m.vm.provider :virtualbox do |vb|
		vb.customize [ 'modifyvm', :id, '--memory', '750' ]
		vb.customize [ 'modifyvm', :id, '--cpus', '1' ]
		vb.customize [ 'modifyvm', :id, '--name', "cluster-node" ]
                vb.customize [ "modifyvm", :id, "--nic2", "hostonly" ]
                vb.customize [ "modifyvm", :id, "--hostonlyadapter2", "vboxnet1" ]
  	end
        m.vm.provision "shell", inline: $nnscript
  end

  config.vm.define "node2" do |m|
  	m.vm.box = "ubuntu/xenial64"
        m.vm.hostname = "node.localhost2"
  	m.vm.provider :virtualbox do |vb|
		vb.customize [ 'modifyvm', :id, '--memory', '750' ]
		vb.customize [ 'modifyvm', :id, '--cpus', '1' ]
		vb.customize [ 'modifyvm', :id, '--name', "cluster-node2" ]
                vb.customize [ "modifyvm", :id, "--nic2", "hostonly" ]
                vb.customize [ "modifyvm", :id, "--hostonlyadapter2", "vboxnet1" ]
  	end
        m.vm.provision "shell", inline: $nn2script
  end
end
