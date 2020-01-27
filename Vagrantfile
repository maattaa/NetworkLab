# -*- mode: ruby -*-
# vi: set ft=ruby :


IMAGE = "bento/ubuntu-16.04"

Vagrant.configure("2") do |config|


	config.vm.provider "virtualbox" do |v|
		v.gui = false
		v.customize ["modifyvm", :id, "--vram", "128"]
	end
	
  config.vm.define "lab1" do |master|
	master.vm.box = IMAGE
	master.vm.hostname ="lab1"
	master.vm.network "private_network", ip: "10.0.0.10", virtualbox__intnet: "intnet1"
	master.vm.network "private_network", ip: "10.0.1.10", virtualbox__intnet: "intnet2"
	sshmaster(master)
	sshslave(master)
  end

    config.vm.define "lab2" do |slave|
	slave.vm.box = IMAGE
	slave.vm.hostname ="lab2"
	slave.vm.network "private_network", ip: "10.0.0.11", virtualbox__intnet: "intnet1"
	sshslave(slave)
  end

    config.vm.define "lab3" do |slave|
	slave.vm.box = IMAGE
	slave.vm.hostname ="lab3"
	slave.vm.network "private_network", ip: "10.0.1.11", virtualbox__intnet: "intnet2"
	sshslave(slave)
  end

config.vm.provision "shell", inline: <<-SHELL
  apt-get install -y avahi-daemon libnss-mdns traceroute lynx xfce4 virtualbox-guest-dkms virtualbox-guest-utils virtualbox-guest-x11
SHELL

config.ssh.insert_key = false
config.ssh.forward_agent = false
config.ssh.private_key_path = ["C:/Users/AMaattanen/.ssh/vagrant", "~/.vagrant.d/insecure_private_key"]
config.vm.provision :shell, :inline =>"
	cat << EOF > /etc/profile.d/xrandr_resolution.sh
	xrandr --size 1280x720
	EOF
	"

#CUSTOM METHODS
def sshmaster(machine)
machine.vm.provision "file", source: "C:/Users/AMaattanen/.ssh/vagrant", destination: "/home/vagrant/.ssh/id_rsa"
machine.vm.provision :shell, :inline =>"
	sudo chmod 600 /home/vagrant/.ssh/id_rsa
	"
end

def sshslave(machine)
machine.vm.provision "file", source: "C:/Users/AMaattanen/.ssh/vagrant.pub", destination: "/home/vagrant/.ssh/id_rsa.pub"
machine.vm.provision :shell, :inline =>"
	cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
	sudo chmod 700 /home/vagrant/.ssh/
	sudo chmod 644 /home/vagrant/.ssh/authorized_keys
	sudo chmod 644 /home/vagrant/.ssh/id_rsa.pub
	"
end

end