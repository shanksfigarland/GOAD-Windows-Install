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

1. Clone `GOAD` to a folder of your liking (recommend being the biggest hard drive you have and not C:)
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
If the error below appears, try disabling any virtualbox network adaptors you had. This will conflict with GOAD.
![a](https://github.com/shanksfigarland/GOAD-Windows-Install/blob/main/vmneterror.png?raw=true)


Wait for installation to end

3. Go to VMWare Workstation's Virtual Network Editor
- Add a new NIC adapter to 192.168.56.0 (Host-only) then add to you Kali and keep your NAT otherwise you won't have internet
![alt text](https://github.com/shanksfigarland/GOAD-Windows-Install/blob/main/nichostonly.png?raw=true)

   
- Install dependencies for `ansible`

```bash
pip install --upgrade pip
pip install ansible-core==2.12.6
pip install pywinrm

sudo apt install sshpass lftp rsync openssh-client
```

- Clone the repo in your KALI machine

Install ansible-galaxy, go to the ansible folder `(GOAD/ansible)` and run:
`ansible-galaxy install -r ansible/requirements.yml`

Then run the install script in the root folder

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

SMB         192.168.56.12   445    MEEREEN          [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:MEEREEN) (domain:essos.local) (signing:True) (SMBv1:True)
SMB         192.168.56.10   445    KINGSLANDING     [*] Windows 10.0 Build 17763 x64 (name:KINGSLANDING) (domain:sevenkingdoms.local) (signing:True) (SMBv1:False)
SMB         192.168.56.22   445    CASTELBLACK      [*] Windows 10.0 Build 17763 x64 (name:CASTELBLACK) (domain:north.sevenkingdoms.local) (signing:False) (SMBv1:False)
SMB         192.168.56.23   445    BRAAVOS          [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:BRAAVOS) (domain:essos.local) (signing:False) (SMBv1:True)
SMB         192.168.56.11   445    WINTERFELL       [*] Windows 10.0 Build 17763 x64 (name:WINTERFELL) (domain:north.sevenkingdoms.local) (signing:True) (SMBv1:False)
```

If these appear, you're good to go.

# Bonus: ELK

1. Comment all boxes and uncomment ELK on Vagrantfile - location: `\ad\GOAD\providers\vmware\Vagrantfile`
```bash
   Vagrant.configure("2") do |config|

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'vmware_desktop'

boxes = [
  # windows server 2022 : don't work for now
  #{ :name => "DC01",  :ip => "192.168.56.10", :box => "StefanScherer/windows_2022", :box_version => "2021.08.23", :os => "windows"},
  # windows server 2019
 # { :name => "GOAD-DC01",  :ip => "192.168.56.10", :box => "StefanScherer/windows_2019", :box_version => "2021.05.15", :os => "windows"},
  # windows server 2019
  #{ :name => "GOAD-DC02",  :ip => "192.168.56.11", :box => "StefanScherer/windows_2019", :box_version => "2021.05.15", :os => "windows"},
  # windows server 2016
  #{ :name => "GOAD-DC03",  :ip => "192.168.56.12", :box => "StefanScherer/windows_2016", :box_version => "2017.12.14", :os => "windows"},
  # windows server 2019
  #{ :name => "SRV01", :ip => "192.168.56.21", :box => "StefanScherer/windows_2019", :box_version => "2020.07.17", :os => "windows"},
  # windows server 2019
 # { :name => "GOAD-SRV02", :ip => "192.168.56.22", :box => "StefanScherer/windows_2019", :box_version => "2020.07.17", :os => "windows"},
  # windows server 2016
  #{ :name => "GOAD-SRV03", :ip => "192.168.56.23", :box => "StefanScherer/windows_2016", :box_version => "2019.02.14", :os => "windows"}
  
  # ELK
 { :name => "GOAD-ELK", :ip => "192.168.56.50", :box => "bento/ubuntu-18.04", :os => "linux",
  :forwarded_port => [
    {:guest => 22, :host => 2210, :id => "ssh"}
  ]
}
]

  config.vm.provider "virtualbox" do |v|
    v.memory = 4000
    v.cpus = 2
  end

  config.vm.provider "vmware_desktop" do |v|
    v.vmx["memsize"] = "4000"
    v.vmx["numvcpus"] = "2"
    # v.force_vmware_license = "workstation"  # force the licence for fix some vagrant plugin issue
  end

  # disable rdp forwarded port inherited from StefanScherer box
  config.vm.network :forwarded_port, guest: 3389, host: 3389, id: "rdp", auto_correct: true, disabled: true

  # no autoupdate if vagrant-vbguest is installed
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.boot_timeout = 600
  config.vm.graceful_halt_timeout = 600
  config.winrm.retry_limit = 30
  config.winrm.retry_delay = 10

  boxes.each do |box|
    config.vm.define box[:name] do |target|
      # BOX
      target.vm.provider "virtualbox" do |v|
        v.name = box[:name]
        v.customize ["modifyvm", :id, "--groups", "/GOAD"]
      end
      target.vm.box_download_insecure = box[:box]
      target.vm.box = box[:box]
      if box.has_key?(:box_version)
        target.vm.box_version = box[:box_version]
      end

      # issues/49
      target.vm.synced_folder '.', '/vagrant', disabled: true

      # IP
      target.vm.network :private_network, ip: box[:ip]

      # OS specific
      if box[:os] == "windows"
        target.vm.guest = :windows
        target.vm.communicator = "winrm"
        target.vm.provision :shell, :path => "../../../../vagrant/Install-WMF3Hotfix.ps1", privileged: false
        target.vm.provision :shell, :path => "../../../../vagrant/ConfigureRemotingForAnsible.ps1", privileged: false

        # fix ip for vmware
        if ENV['VAGRANT_DEFAULT_PROVIDER'] == "vmware_desktop"
          target.vm.provision :shell, :path => "../../../../vagrant/fix_ip.ps1", privileged: false, args: box[:ip]
        end

      else
        target.vm.communicator = "ssh"
      end

      if box.has_key?(:forwarded_port)
        # forwarded port explicit
        box[:forwarded_port] do |forwarded_port|
          target.vm.network :forwarded_port, guest: forwarded_port[:guest], host: forwarded_port[:host], host_ip: "127.0.0.1", id: forwarded_port[:id]
        end
      end

    end
  end
end
   ```
2. `vagrant up` to install ELK
When it finishes, revert to the original file again:
```bash
Vagrant.configure("2") do |config|

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'vmware_desktop'

boxes = [
  # windows server 2022 : don't work for now
  #{ :name => "DC01",  :ip => "192.168.56.10", :box => "StefanScherer/windows_2022", :box_version => "2021.08.23", :os => "windows"},
  # windows server 2019
  { :name => "GOAD-DC01",  :ip => "192.168.56.10", :box => "StefanScherer/windows_2019", :box_version => "2021.05.15", :os => "windows"},
  # windows server 2019
  { :name => "GOAD-DC02",  :ip => "192.168.56.11", :box => "StefanScherer/windows_2019", :box_version => "2021.05.15", :os => "windows"},
  # windows server 2016
  { :name => "GOAD-DC03",  :ip => "192.168.56.12", :box => "StefanScherer/windows_2016", :box_version => "2017.12.14", :os => "windows"},
  # windows server 2019
  #{ :name => "SRV01", :ip => "192.168.56.21", :box => "StefanScherer/windows_2019", :box_version => "2020.07.17", :os => "windows"},
  # windows server 2019
  { :name => "GOAD-SRV02", :ip => "192.168.56.22", :box => "StefanScherer/windows_2019", :box_version => "2020.07.17", :os => "windows"},
  # windows server 2016
  { :name => "GOAD-SRV03", :ip => "192.168.56.23", :box => "StefanScherer/windows_2016", :box_version => "2019.02.14", :os => "windows"}
  
  # ELK
# { :name => "GOAD-ELK", :ip => "192.168.56.50", :box => "bento/ubuntu-18.04", :os => "linux",
#   :forwarded_port => [
#     {:guest => 22, :host => 2210, :id => "ssh"}
#   ]
# }
]

  config.vm.provider "virtualbox" do |v|
    v.memory = 4000
    v.cpus = 2
  end

  config.vm.provider "vmware_desktop" do |v|
    v.vmx["memsize"] = "4000"
    v.vmx["numvcpus"] = "2"
    # v.force_vmware_license = "workstation"  # force the licence for fix some vagrant plugin issue
  end

  # disable rdp forwarded port inherited from StefanScherer box
  config.vm.network :forwarded_port, guest: 3389, host: 3389, id: "rdp", auto_correct: true, disabled: true

  # no autoupdate if vagrant-vbguest is installed
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.boot_timeout = 600
  config.vm.graceful_halt_timeout = 600
  config.winrm.retry_limit = 30
  config.winrm.retry_delay = 10

  boxes.each do |box|
    config.vm.define box[:name] do |target|
      # BOX
      target.vm.provider "virtualbox" do |v|
        v.name = box[:name]
        v.customize ["modifyvm", :id, "--groups", "/GOAD"]
      end
      target.vm.box_download_insecure = box[:box]
      target.vm.box = box[:box]
      if box.has_key?(:box_version)
        target.vm.box_version = box[:box_version]
      end

      # issues/49
      target.vm.synced_folder '.', '/vagrant', disabled: true

      # IP
      target.vm.network :private_network, ip: box[:ip]

      # OS specific
      if box[:os] == "windows"
        target.vm.guest = :windows
        target.vm.communicator = "winrm"
        target.vm.provision :shell, :path => "../../../../vagrant/Install-WMF3Hotfix.ps1", privileged: false
        target.vm.provision :shell, :path => "../../../../vagrant/ConfigureRemotingForAnsible.ps1", privileged: false

        # fix ip for vmware
        if ENV['VAGRANT_DEFAULT_PROVIDER'] == "vmware_desktop"
          target.vm.provision :shell, :path => "../../../../vagrant/fix_ip.ps1", privileged: false, args: box[:ip]
        end

      else
        target.vm.communicator = "ssh"
      end

      if box.has_key?(:forwarded_port)
        # forwarded port explicit
        box[:forwarded_port] do |forwarded_port|
          target.vm.network :forwarded_port, guest: forwarded_port[:guest], host: forwarded_port[:host], host_ip: "127.0.0.1", id: forwarded_port[:id]
        end
      end

    end
  end
end
```
3. Go to your Kali > GOAD folder and edit `goad.sh` and add `elk.yml` to `ANSIBLE_PLAYBOOKS`
```bash
ANSIBLE_PLAYBOOKS="build.yml ad-servers.yml ad-parent_domain.yml ad-child_domain.yml ad-members.yml ad-trusts.yml ad-data.yml ad-gmsa.yml laps.yml ad-relations.yml adcs.yml ad-acl.yml servers.yml security.yml vulnerabilities.yml reboot.yml elk.yml"
```
4. Run the install again with the following:
```bash
./goad.sh -t install -l GOAD -p vmware -m local -a -r elk.yml
```
5. Vagrant up
6. Access your ELK at `http://192.168.56.50:5601/`

Have fun!
