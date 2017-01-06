# -*- mode: ruby -*-
# vi: set ft=ruby :
$script = <<SCRIPT

echo Installing dependencies...
sudo apt-get update
sudo apt-get install -y unzip curl

echo Fetching Consul...
cd /tmp/
curl -s https://releases.hashicorp.com/consul/0.7.2/consul_0.7.2_linux_amd64.zip -o consul.zip

echo Installing Consul...
unzip consul.zip
sudo chmod +x consul
sudo mv consul /usr/bin/consul

sudo mkdir /etc/consul.d
sudo chmod a+w /etc/consul.d

SCRIPT

$master_script = <<SCRIPT

echo Configuring as cluster master...
consul agent -server -bootstrap-expect=1 \
    -data-dir=/tmp/consul -node=agent-one -bind=172.20.20.10 \
    -config-dir=/etc/consul.d &

SCRIPT

$client_script = <<SCRIPT

echo Configuring as cluster client...
consul agent -data-dir=/tmp/consul -node=agent-two \
    -bind=172.20.20.11 -config-dir=/etc/consul.d \
    -join=172.20.20.10 &

SCRIPT

# Specify a custom Vagrant box for the demo
DEMO_BOX_NAME =  ENV['DEMO_BOX_NAME'] || "debian/jessie64"

# Vagrantfile API/syntax version.
# NB: Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = DEMO_BOX_NAME

  config.vm.provision "shell", inline: $script, run: "once"

  config.vm.define "n1" do |n1|
      n1.vm.hostname = "n1"
      n1.vm.network "private_network", ip: "172.20.20.10"
      n1.vm.provision "shell", inline: $master_script
  end

  config.vm.define "n2" do |n2|
      n2.vm.hostname = "n2"
      n2.vm.network "private_network", ip: "172.20.20.11"
      n2.vm.provision "shell", inline: $client_script
  end
end