# Installation process

## Pre-requisites
- Windows (HOST)
- VMWARE Workstation 
- Vagrant (Windows)

## VMware
- Your Kali with 2 NICs (1 NAT - 1 vmnet 192.168.56.0)

## Install VMware Workstation 
https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html

## Install Vagrant for Windows
https://developer.hashicorp.com/vagrant/install?product_intent=vagrant#Windows

## Install Vagrant plugin 
In the same folder your vagrant.exe is, open a new cmd and type:
```bash
vagrant plugin install vagrant-vmware-desktop
```

## Install git bash for Windows
https://git-scm.com/download/win

1. Clone *GOAD* to a folder of your liking (recommend being the biggest hard drive you have and not C:)
- right click on the folder > `Open Git Bash here` and type:
```bash
git clone https://github.com/Orange-Cyberdefense/GOAD
```
2. Go to provider folder
```bash
cd GOAD/ad/GOAD/providers/vmware
```
type:
```bash
vagrant up
```
Wait for installation to end

3. Go to your kali 
3.1 Install dependencies for `ansible`
```bash
pip install --upgrade pip
pip install ansible-core==2.12.6
pip install pywinrm

sudo apt install sshpass lftp rsync openssh-client
```
3.2 Clone the repo again and run the install script
```bash
./goad.sh -t install -l GOAD -p vmware -m local -a
```
Wait to end, grab a coffee, it's gonna take a while.



