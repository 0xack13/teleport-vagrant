# -*- mode: ruby -*-
# vi: set ft=ruby :

$initiatescript = <<SHELL
sudo cp /mnt/teleport-v6.2.7-linux-amd64-bin.tar.gz ~/
cd ~/
tar -xzf teleport-v6.2.7-linux-amd64-bin.tar.gz
cd ~/teleport
sudo ./install
sudo mkdir -p /var/lib/teleport
SHELL

$genmaster = <<SHELL
echo "Enable teleport systemd"
cd ~/teleport/examples/systemd
sudo cp teleport.service /etc/systemd/system/teleport.service
sudo systemctl daemon-reload
sudo systemctl enable teleport
sudo systemctl start teleport
sleep 5
sudo tctl auth export --type=tls > /mnt/ca.cert
sudo tctl auth export > /mnt/auth.pub
sudo tctl users add $USER root > /mnt/invitevagrant.txt 
sudo tctl nodes add --ttl=5m --roles=node | sed -n '/teleport/,/3025/p' | sed 's/> //' | sed 's/=.*:3025/=172.20.20.20:3025/g' > /mnt/addnode.sh
SHELL

$gennode = <<SHELL
sudo mkdir -p /var/lib/teleport
sudo cp /mnt/ca.cert /var/lib/teleport/ca.cert
sudo cp /mnt/auth.pub /var/lib/teleport/auth.pub
sudo cp /mnt/addnode.sh /root/
sudo chmod a+x /root/addnode.sh
nohup /bin/sh /root/addnode.sh > /dev/null 2>&1 &
SHELL

system("
    if [ #{ARGV[0]} = 'up' ]; then
        echo 'Downloading teleport..'
        wget -c https://get.gravitational.com/teleport-v6.2.7-linux-amd64-bin.tar.gz
    fi
")

image = "ubuntu/xenial64"
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vagrant.plugins = "vagrant-hostsupdater"
  
  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
  end 
  
  config.vm.box = image
  config.vm.synced_folder ".", "/mnt"
  config.vm.provision "shell", inline: $initiatescript

  config.vm.define "g" do |g|
      g.vm.hostname = "g"
      g.vm.network "private_network", ip: "172.20.20.20"
      g.vm.provision "shell", inline: $genmaster
      g.hostsupdater.aliases = ["g"]
  end

  config.vm.define "n1" do |n1|
      n1.vm.hostname = "n1"
      n1.vm.network "private_network", ip: "172.20.20.21"
      n1.vm.provision "shell", inline: $gennode
  end
  config.vm.define "n2" do |n2|
      n2.vm.hostname = "n2"
      n2.vm.network "private_network", ip: "172.20.20.22"
      n2.vm.provision "shell", inline: $gennode
  end
end
