# -*- mode: ruby -*-
# vi: set ft=ruby :

#Script to set KB layout to Finnish.
$kb = <<-SCRIPT
printf '%s\n' 'XKBMODEL="pc105"' 'XKBLAYOUT="fi"' 'XKBVARIANT=""' 'XKBOPTIONS=""' '' 'BACKSPACE="guess"' | tee /etc/default/keyboard
SCRIPT

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
	hosts(master)
	adduserslab1(master)
	nfs(master)
	nfsserver(master)
	smbserver(master)
	sshfs(master) #sshfs needs to be configured only on clients
	webdav(master)

#	bind9(master)
#	ns1(master)
#	x(master)
#	nginx(master) #has to be manually started
  end

    config.vm.define "lab2" do |slave|
	slave.vm.box = IMAGE
	slave.vm.hostname ="lab2"
	slave.vm.network "private_network", ip: "10.0.0.11", virtualbox__intnet: "intnet1"
	sshslave(slave)
	hosts(slave)
	adduserslab2(slave)
	nfs(slave)
	nfsclient(slave)
	smbclient(slave)
	sshfs(slave) 
	webdav(slave)


#	apache(slave)
#	bind9(slave)
#	ns2(slave)
  end

    config.vm.define "lab3" do |slave|
	slave.vm.box = IMAGE
	slave.vm.hostname ="lab3"
	slave.vm.network "private_network", ip: "10.0.1.11", virtualbox__intnet: "intnet2"
	sshslave(slave)
#	nodejs(slave) #has to be manually started

  end

config.vm.provision "shell", inline: <<-SHELL
	apt-get update
	apt-get install -y traceroute lynx cadaver elinks #avahi-daemon libnss-mdns 
SHELL

config.ssh.insert_key = false
config.ssh.forward_agent = false
config.ssh.private_key_path = ["C:/Users/AMaattanen/.ssh/vagrant", "~/.vagrant.d/insecure_private_key"]

#CUSTOM METHODS#

#WEEK1

#lab1 needs to be able to ssh into sshslaves
def sshmaster(machine)
machine.vm.provision "file", source: "C:/Users/AMaattanen/.ssh/vagrant", destination: "/home/vagrant/.ssh/id_rsa"
machine.vm.provision :shell, :inline =>"
	sudo chmod 600 /home/vagrant/.ssh/id_rsa
	"
end

#other labs need to accept the ssh using sshmasters key
def sshslave(machine)
machine.vm.provision "file", source: "C:/Users/AMaattanen/.ssh/vagrant.pub", destination: "/home/vagrant/.ssh/id_rsa.pub"
machine.vm.provision :shell, :inline =>"
	cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
	sudo chmod 700 /home/vagrant/.ssh/
	sudo chmod 644 /home/vagrant/.ssh/authorized_keys
	sudo chmod 644 /home/vagrant/.ssh/id_rsa.pub
	"
end

#WEEK2
	def x(machine) #set x11 screen resolution higher
		machine.vm.provision "shell", :inline => "apt-get install -y xfce4 virtualbox-guest-dkms virtualbox-guest-utils virtualbox-guest-x11"
		machine.vm.provision "shell", :inline => "echo 'xrandr --size 1280x720' >> /home/vagrant/.profile"
		#set layout to finnish
		machine.vm.provision "shell", :inline => $kb
		#reboot to take effect of keyboard change, this will mess vagrant
		#WILL MESS UP VAGRANT, BOOT MANUALLY TO USE FINNISH KB IN X
		#machine.vm.provision "shell", :inline => "sudo reboot"
	end
	
	def apache(machine) #apache2 for lab2
		machine.vm.provision "shell", inline: <<-SHELL 
		apt-get install -y apache2
		printf "%s\t" "127.0.0.1" "maattanen.com" | tee -a /etc/hosts
		openssl req -x509 -out maattanen.crt -keyout maattanen.key \
		-newkey rsa:2048 -nodes -sha256 \
		-subj '/CN=maattanen.com' -extensions EXT -config <( \
		printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
		cp -f /vagrant/apache.conf /etc/apache2/sites-available/maattanen.conf
		a2ensite maattanen
		a2enmod ssl userdir rewrite
		mkdir -p /home/vagrant/public_html/secure_secrets/ #create 
		cp -f /vagrant/htaccess /home/vagrant/public_html/secure_secrets/.htaccess
		touch /home/vagrant/public_html/secure_secrets/moi
		chmod 775 -R /home/vagrant/public_html
		chown -R $USER:$USER /home/vagrant/public_html
		service apache2 restart
		SHELL
	end

	def nodejs(machine) #nodejs for lab3
		machine.vm.provision "shell", inline: "apt-get install -y nodejs"
#RUNNING THE MACHINE HAS TO BE DONE BY HAND, VAGRANT DOESNT LIKE IT
#		machine.vm.provision "shell", inline: "nodejs /vagrant/helloworld.js"
	end

	def nginx(machine) #nginx for lab1
		machine.vm.provision "shell", inline: "apt-get update; apt-get install -y nginx" #will get error without apt-get update
		machine.vm.provision "shell", inline: "cp /vagrant/nginx.conf /etc/nginx/sites-available/default"
		machine.vm.provision "shell", inline: "chown root:root /etc/nginx/sites-available/default"
		machine.vm.provision "shell", inline: "chmod 744 /etc/nginx/sites-available/default"
#		machine.vm.provision "shell", inline: "rm /etc/nginx/sites-available/*"
#		machine.vm.provision "shell", inline: "cp /vagrant/nginx.conf /etc/nginx/sites-available/lab1.conf"
#STARTING THE NGINX HAS TO BE DONE BY HAND, VAGRANT DOESNT LIKE IT
#		machine.vm.provision "shell", inline: "nginx"
	end
	
	
	#WEEK3
	def ns1(machine)
		machine.vm.provision "shell", inline: <<-SHELL
		printf "%s\n" "" "127.0.0.1	ns1" | tee -a /etc/hosts
		printf "%s\n" "" "10.0.0.11	ns2" | tee -a /etc/hosts
		cp /vagrant/ns1.resolv.conf /etc/resolv.conf
		cp /vagrant/master.named.conf.options /etc/bind/named.conf.options
		cp /vagrant/master.named.conf /etc/bind/named.conf
		cp /vagrant/master.db.insec /etc/bind/db.insec
		cp /vagrant/master.dbrev.insec /etc/bind/dbrev.insec
		service bind9 restart
		SHELL
	end
	
	def ns2(machine)
		machine.vm.provision "shell", inline: <<-SHELL
		printf "%s\n" "" "127.0.0.1	ns2" | tee -a /etc/hosts
		printf "%s\n" "" "10.0.0.10	ns1" | tee -a /etc/hosts
		cp /vagrant/ns2.resolv.conf /etc/resolv.conf
		cp /vagrant/slave.named.conf /etc/bind/named.conf
		cp /vagrant/master.not.db.insec /etc/bind/master.not.insec
		cp /vagrant/master.not.dbrev.insec /etc/bind/master.dbrev.insec
		SHELL
	end	
	
	def bind9(machine)
		machine.vm.provision "shell", inline: <<-SHELL
		apt-get install -y bind9
		SHELL
	end
	
	
	#WEEK4
	def hosts(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	echo "10.0.0.10	lab1" >> /etc/hosts
	echo "10.0.0.11	lab2" >> /etc/hosts
	SHELL
	end
	
	def adduserslab1(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	adduser testuser1 --uid 1001 --disabled-password --gecos ""
	touch /home/testuser1/test.txt
	echo ":)" >> /home/testuser1/test.txt
	sudo echo "testuser1:testuser" | sudo chpasswd
	adduser testuser2 --uid 1002 --disabled-password --gecos "" --no-create-home 
	sudo echo "testuser2:testuser" | sudo chpasswd
	chown testuser1:testuser1 -R /home/testuser1
	chmod 700 -R /home/testuser1
	SHELL
	end
	
	def adduserslab2(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	adduser testuser1 --uid 1001 --disabled-password --gecos "" --no-create-home 
	sudo echo "testuser1:testuser" | sudo chpasswd
	adduser testuser2 --uid 1002 --disabled-password --gecos "" --no-create-home 
	sudo echo "testuser2:testuser" | sudo chpasswd
	SHELL
	end
	
	def nfs(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	apt-get install -y nfs-kernel-server
	SHELL
	end
	
	def nfsserver(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	printf "%s\n" "/home/testuser1 lab2(rw,no_subtree_check,sync)" | tee -a /etc/exports
	echo "	portmap:ALL
	lockd:ALL
	mountd:ALL
	rquotad:ALL
	statd:ALL" >> /etc/hosts.deny
	echo "	portmap: lab2
	lockd: lab2
	mountd: lab2
	rquotad: lab2
	statd: lab2" >> /etc/hosts.allow
	exportfs -a
	systemctl restart nfs-kernel-server
	SHELL
	end
	
	def nfsclient(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	mkdir -p /home/testuser1
	mount lab1:/home/testuser1 /home/testuser1
	SHELL
	end
	
	def smbserver(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	apt-get install -y samba
	cp /vagrant/smb.conf /etc/samba/
	systemctl restart smbd nmbd
	#REMEMBER TO
	#sudo smbpasswd -a testuser1
	SHELL
	end
	
	def smbclient(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	apt-get install -y cifs-utils
	#mount -t cifs -o username=testuser1,password=testuser //lab1/testuser1 /home/testuser1/
	SHELL
	end
	
	
	def sshfs(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	mkdir -p /home/testuser1/mnt
	chown testuser1:testuser1 -R /home/testuser1
	chmod 700 -R /home/testuser1
	apt-get install -y sshfs
	#sudo su testuser1
	#sshfs -o reconnect lab1:/home/testuser1 /home/testuser1/mnt

	SHELL
	end
	
	def webdav(machine)
	machine.vm.provision "shell", inline: <<-SHELL
	apt-get update
	apt-get install -y apache2 apache2-utils
	a2enmod dav*
	service apache2 restart
	a2enmod auth_digest
	service apache2 restart
	mkdir -p /var/www/WebDAV/files
	touch /var/www/WebDAV/files/test
	chown www-data:vagrant -R /var/www/WebDAV
	chmod 770 -R /var/www/WebDAV
	cp /vagrant/webdav.conf /etc/apache2/sites-available/000-default.conf
	# htdigest -c /var/www/WebDAV/.digest testuser1 testuser1
	# chown www-data:root /var/www/WebDAV/.digest
	# chmod 700 /var/www/WebDAV/.digest
	# service apache2 restart
	SHELL
	end
	

end