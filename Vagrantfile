$master_script = <<SCRIPT
#!/bin/bash

apt-get install curl -y
REPOCM=${REPOCM:-cm5}
CM_REPO_HOST=${CM_REPO_HOST:-archive.cloudera.com}
CM_MAJOR_VERSION=$(echo $REPOCM | sed -e 's/cm\\([0-9]\\).*/\\1/')
CM_VERSION=$(echo $REPOCM | sed -e 's/cm\\([0-9][0-9]*\\)/\\1/')
OS_CODENAME=$(lsb_release -sc)
OS_DISTID=$(lsb_release -si | tr '[A-Z]' '[a-z]')
if [ $CM_MAJOR_VERSION -ge 4 ]; then
  cat > /etc/apt/sources.list.d/cloudera-$REPOCM.list <<EOF
deb [arch=amd64] http://$CM_REPO_HOST/cm$CM_MAJOR_VERSION/$OS_DISTID/$OS_CODENAME/amd64/cm $OS_CODENAME-$REPOCM contrib
deb-src http://$CM_REPO_HOST/cm$CM_MAJOR_VERSION/$OS_DISTID/$OS_CODENAME/amd64/cm $OS_CODENAME-$REPOCM contrib
EOF
curl -s http://$CM_REPO_HOST/cm$CM_MAJOR_VERSION/$OS_DISTID/$OS_CODENAME/amd64/cm/archive.key > key
apt-key add key
rm key
fi
add-apt-repository -y ppa:webupd8team/java
echo debconf shared/accepted-oracle-license-v1-1 select true | debconf-set-selections
echo debconf shared/accepted-oracle-license-v1-1 seen true | debconf-set-selections
apt-get update
export DEBIAN_FRONTEND=noninteractive
apt-get -q -y --force-yes install oracle-java8-installer
apt-get -q -y --force-yes install cloudera-manager-server-db cloudera-manager-server cloudera-manager-daemons
service cloudera-scm-server-db initdb
service cloudera-scm-server-db start
service cloudera-scm-server start
SCRIPT

$hosts_script = <<SCRIPT
cat > /etc/hosts <<EOF
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

EOF
SCRIPT

$sshd_script = <<SCRIPT
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
service sshd reload
SCRIPT

Vagrant.configure("2") do |config|

  # Install required plugins
  required_plugins = %w( vagrant-hostmanager vagrant-disksize )
  _retry = false
  required_plugins.each do |plugin|
    unless Vagrant.has_plugin? plugin
      system "vagrant plugin install #{plugin}"
      _retry=true
    end
  end

  if (_retry)
    exec "vagrant " + ARGV.join(' ')
  end

  # Define base image
  config.vm.box = "ubuntu/xenial64"

  # Set the disk size
  config.disksize.size = "25GB"

  # Manage /etc/hosts on host and VMs
  config.hostmanager.enabled = false
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true
  config.hostmanager.ignore_private_ip = false

  config.vm.define :master do |master|
    master.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-node1"
      v.customize ["modifyvm", :id, "--memory", "6144"]
    end
    master.vm.network :private_network, ip: "10.211.55.100"
    master.vm.hostname = "vm-cluster-node1.example.org"
    master.vm.provision :shell, :inline => $hosts_script
    master.vm.provision :hostmanager
    master.vm.provision :shell, :inline => $master_script
    master.vm.provision :shell, :inline => $sshd_script
  end

  config.vm.define :slave1 do |slave1|
    slave1.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-node2"
      v.customize ["modifyvm", :id, "--memory", "2048"]
    end
    slave1.vm.network :private_network, ip: "10.211.55.101"
    slave1.vm.hostname = "vm-cluster-node2.example.org"
    slave1.vm.provision :shell, :inline => $hosts_script
    slave1.vm.provision :hostmanager
    slave1.vm.provision :shell, :inline => $sshd_script
  end

  config.vm.define :slave2 do |slave2|
    slave2.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-node3"
      v.customize ["modifyvm", :id, "--memory", "2048"]
    end
    slave2.vm.network :private_network, ip: "10.211.55.102"
    slave2.vm.hostname = "vm-cluster-node3.example.org"
    slave2.vm.provision :shell, :inline => $hosts_script
    slave2.vm.provision :hostmanager
    slave2.vm.provision :shell, :inline => $sshd_script
  end

  config.vm.define :slave3 do |slave3|
    slave3.vm.provider :virtualbox do |v|
      v.name = "vm-cluster-node4"
      v.customize ["modifyvm", :id, "--memory", "2048"]
    end
    slave3.vm.network :private_network, ip: "10.211.55.103"
    slave3.vm.hostname = "vm-cluster-node4.example.org"
    slave3.vm.provision :shell, :inline => $hosts_script
    slave3.vm.provision :hostmanager
    slave3.vm.provision :shell, :inline => $sshd_script
  end

  config.vm.define :freeipa do |freeipa|
    freeipa.vm.provider :virtualbox do |v|
      v.name = "freeipa"
      v.customize ["modifyvm", :id, "--memory", "1024"]
    end
    freeipa.vm.box = "vStone/centos-7.x-puppet.3.x"
    freeipa.vm.network :private_network, ip: "10.211.55.200"
    freeipa.vm.hostname = "freeipa.example.org"
    freeipa.vm.provision :shell, :inline => $hosts_script
    freeipa.vm.provision :hostmanager
    freeipa.vm.provision :shell, :inline => $sshd_script
    # And install freeipa:
    freeipa.vm.provision :shell, :inline => "puppet module install huit/ipa"
    freeipa.vm.provision "puppet" do |puppet|
	  puppet.manifest_file = "default.pp"
	end
  end

end