# Installation of GOAD for Windows

Main project by Mayfly: https://github.com/Orange-Cyberdefense/GOAD

This was fully tested only on a Windows 10 machine with 64GB of RAM. 

This may also work for `GOAD-light` and `NHA`.


## Pre-requisites
- Windows 10 (HOST)
- VMWARE Workstation 
- Vagrant (Windows)
- Git bash (Windows)
- Kali (inside VMWARE)
- A lot of disk (200GB)
- A lot of ram (At least 32GB)

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
git clone https://github.com/Orange-Cyberdefense/GOAD
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
- Add a new NIC adapter to 192.168.56.0
- Keep your NAT otherwise you won't have internet
   
- Install dependencies for `ansible`

```bash
pip install --upgrade pip
pip install ansible-core==2.12.6
pip install pywinrm

sudo apt install sshpass lftp rsync openssh-client
```
- Clone the repo again and run the install script

```bash
./goad.sh -t install -l GOAD -p vmware -m local -a
```
Wait to end, grab a coffee, it's gonna take a while.

When it ends this will appear:
```bash
[✓] Command successfully executed
[✓] your lab is successfully setup ! have fun ;)
```

To test if all machines are working go to your Kali and use netexec to test
```bash
nxc smb 192.168.56.0/24
```
If 5 machines (MEEREEN, KINGSLANDING, CASTELBLACK, BRAAVOS, WINTERFELL) shows up, you're good to go.



