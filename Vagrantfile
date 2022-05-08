#Modified version of the REF.script-base.txt file for compatibility with Ubuntu version 20.04 and newer Docker repositories.
#Author: Jean Queiroz (https://github.com/jcqueiroz)
$install_docker_script = <<SCRIPT
echo Installing Docker...
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt update
sudo apt install docker-ce -y
sudo service docker restart
sudo systemctl status docker
sudo systemctl enable docker
sudo service docker restart
sudo usermod -aG docker ubuntu
sudo usermod -aG docker vagrant
sudo ufw disable
systemctl restart docker
sudo chmod 666 /var/run/docker.sock
SCRIPT

$manager_script = <<SCRIPT
echo Swarm Init...
sudo docker swarm init --listen-addr 192.168.56.1:2377 --advertise-addr 192.168.56.1:2377
sudo docker swarm join-token --quiet worker > /vagrant/worker_token
SCRIPT

$worker_script = <<SCRIPT
echo Swarm Join...
sudo docker swarm join --token $(cat /vagrant/worker_token) 192.168.56.1:2377
SCRIPT

Vagrant.configure('2') do |config|
vm_box = 'ubuntu/focal64'
config.vm.define :manager, primary: true  do |manager|
    manager.vm.box = vm_box
    manager.vm.box_check_update = true
    manager.vm.network :private_network, ip: "192.168.56.1"
    manager.vm.network :forwarded_port, guest: 8080, host: 8080
    manager.vm.network :forwarded_port, guest: 5000, host: 5000
    manager.vm.hostname = "manager"
    manager.vm.synced_folder ".", "/vagrant"
    manager.vm.provision "shell", inline: $install_docker_script, privileged: true
    manager.vm.provision "shell", inline: $manager_script, privileged: true
    manager.vm.provider "virtualbox" do |vb|
      vb.name = "manager"
      vb.memory = "1024"
    end
  end
(1..3).each do |i|
    config.vm.define "worker0#{i}" do |worker|
      worker.vm.box = vm_box
      worker.vm.box_check_update = true
      worker.vm.network :private_network, ip: "192.168.56.10#{i}"
      worker.vm.hostname = "worker0#{i}"
      worker.vm.synced_folder ".", "/vagrant"
      worker.vm.provision "shell", inline: $install_docker_script, privileged: true
      worker.vm.provision "shell", inline: $worker_script, privileged: true
      worker.vm.provider "virtualbox" do |vb|
        vb.name = "worker0#{i}"
        vb.memory = "1024"
      end
    end
  end
end