$install_docker_script = <<SCRIPT
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker ubuntu
SCRIPT
$master_script = <<SCRIPT
echo Swarm Init...
sudo docker swarm init --listen-addr 10.0.0.1:1337 --advertise-addr 10.0.0.1:1337
sudo docker swarm join-token --quiet slave > /vagrant/slave_token
SCRIPT
$slave_script = <<SCRIPT
echo Swarm Join...
sudo docker swarm join --token $(cat /vagrant/slave_token) 10.0.0.1:1337
SCRIPT
Vagrant.configure('2') do |config|
vm_box = 'ubuntu/xenial64'
config.vm.define :master, primary: true  do |master|
    master.vm.box = vm_box
    master.vm.box_check_update = true
    master.vm.network :private_network, ip: "10.0.0.1"
    master.vm.network :forwarded_port, guest: 8080, host: 8080
    master.vm.network :forwarded_port, guest: 5000, host: 5000
    master.vm.hostname = "master"
    master.vm.synced_folder ".", "/vagrant"
    master.vm.provision "shell", inline: $install_docker_script, privileged: true
    master.vm.provision "shell", inline: $master_script, privileged: true
    master.vm.provider "virtualbox" do |vb|
      vb.name = "master"
      vb.memory = "1024"
    end
  end
(2..3).each do |i|
    config.vm.define "slave0#{i}" do |slave|
      slave.vm.box = vm_box
      slave.vm.box_check_update = true
      slave.vm.network :private_network, ip: "10.0.0.#{i}"
      slave.vm.hostname = "slave0#{i}"
      slave.vm.synced_folder ".", "/vagrant"
      slave.vm.provision "shell", inline: $install_docker_script, privileged: true
      slave.vm.provision "shell", inline: $slave_script, privileged: true
      slave.vm.provider "virtualbox" do |vb|
        vb.name = "slave0#{i}"
        vb.memory = "1024"
      end
    end
  end
end