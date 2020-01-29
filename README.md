# CS-E4160 - Laboratory Works in Networking and Security

These are assignments made during to course. Each week we are configuring VM's for different purposes. Most of the configuring is done through Vagrantfile, however I might do some parts manually inside the machine. The assignments are not here.

## Getting Started

This is the Vagrantfile needed to provision Vagrant.

### Prerequisites

What things you need to install the software and how to install them

* [Vagrant](https://www.vagrantup.com/downloads.html) - The tool used to run the VM's
* [VirtualBox (ships with Vagrant)](https://www.virtualbox.org) - Provisioner for the VM's

### Installing

After installing Vagrant, running "vagrant init" from Vagrant's location will create Vagrantfile. After that you can 


## Running the file

After Vagrantfile is in place, you can start the machines by

```
vagrant up --provision
```

To get rid of the machines (running or not), you can 
```
vagrant destroy -f
```

## Versioning

Different week assignments are done through different forks. 

