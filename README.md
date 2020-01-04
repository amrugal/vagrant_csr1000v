# vagrant_csr1000v
Spinning csr1000v router using vagrant with virtualbox as provisioner. 

What we need:
  - csr1000v iso image.
  - installed vagrant
  - installed virtual box

Step 1:
  #In virtualbox gui click new machine.\
  #Set Name: csr1000v (for example). \
  #Set Type: Linux. \
  #Set Version: Other Linux (64-bit). \
  \
  #Choose Create virtual hard disk now > VDI (VIrtualBox Disk Image) > Fixed size > Default disk name and location > Size 8GB.\
  \
  #Right click on created csr machine > Settings... > Serial Ports > Thick Enable Serial Port > Port Mode: Disconnected. \
  \
  #Start your machine and point to csr iso image if asked and not mounted before spinning it up in Settings > Storage. \
  \
  #CSR1000v boot time can be long, 15 minutes for example. \
  \
  #Shut it down and be sure the iso image is unmounted from optical drive. \
  \
Step 2: 
  Make new directories, one for vagrant project and second for storing vagrant package Vagrant/Boxes for package. 
  ```bash
    mkdir /home/adrian/Vagrant/
    mkdir /home/adrian/Vagrant/Boxes/
  ```
  Type in shell: 
  ```bash
    vagrant package --base csr1000v --output /home/adrian/Vagrant/Boxes/
  ```
  output: 
  ```bash
    ==> csr1000v: Exporting VM...
    ==> csr1000v: Compressing package to: /home/adrian/Vagrant/boxes/cisco-csr1000v
  ```
  (csr1000v is your machine name created in VirtualBox via GUI) 

  We can check file presence and size: 
  ```bash
    ll -h /home/adrian/Vagrant/boxes/ 
  ```
  Vagrant package named cisco-csr1000v should be created: 
  ```bash
    total 1.1G
    drwxrwxr-x 2 adrian adrian 4.0K Jan  3 20:22 ./
    drwxrwxr-x 4 adrian adrian 4.0K Jan  3 22:45 ../
    -rw-rw-r-- 1 adrian adrian 1.1G Jan  3 20:24 cisco-csr1000v
  ```
   Then type: 
  ```bash     
    vagrant box add /home/adrian/Vagrant/boxes/cisco-csr1000v --name cisco-csr1000v-box
  ```
  output: 
  ```bash
    ==> box: Box file was not detected as metadata. Adding it directly...
    ==> box: Adding box 'cisco-csr1000b-box' (v0) for provider: 
        box: Unpacking necessary files from: file:///home/adrian/Vagrant/boxes/cisco-csr1000v
    ==> box: Successfully added box 'cisco-csr1000b-box' (v0) for 'virtualbox'!
  ```
  ```bash
    vagrant box list
  ```
  output: 
  ```bash
    cisco-csr1000v-box (virtualbox, 0)
  ```
  (vagrant box should be created)

Step 3: 
    Now we need to create "Vagrantfile" in our project directory by creating it in any text editor or by typing in shell:
    vagrant init
    then edit Vagrantfile in nano for example. 
```ruby
Vagrant.configure("2") do |config|
  
  config.vm.define "rtr1" do |rtr1|
    rtr1.vm.box = "cisco-csr1000v-box" 
    rtr1.vm.boot_timeout = 1 
    rtr1.vm.hostname = "rtr1" #new csr name in VirtualBox instance
    rtr1.vm.network :forwarded_port, guest: 22, host: 2201, id: "ssh" 
    rtr1.vm.network "public_network",use_dhcp_assigned_default_route: true, bridge: "enp0s3: Ethernet"
    rtr1.vm.network "private_network", ip: "192.168.100.1"
    rtr1.vm.network "private_network",  ip: "192.168.200.1"
    rtr1.vm.provider "virtualbox" do |vb|
      vb.name = "rtr1"
      vb.memory = "3072"
      vb.cpus = 2
    end
  end
end
```
Step 4: 
  Type: 
```bash
vagrant up
```
output: 
```bash
Bringing machine 'rtr1' up with 'virtualbox' provider...
==> rtr1: Importing base box 'cisco-csr1000b-box'...
==> rtr1: Matching MAC address for NAT networking...
==> rtr1: Setting the name of the VM: rtr1
==> rtr1: Clearing any previously set network interfaces...
==> rtr1: Preparing network interfaces based on configuration...
    rtr1: Adapter 1: nat
    rtr1: Adapter 2: bridged
    rtr1: Adapter 3: hostonly
    rtr1: Adapter 4: hostonly
==> rtr1: Forwarding ports...
    rtr1: 22 (guest) => 2201 (host) (adapter 1)
==> rtr1: Running 'pre-boot' VM customizations...
==> rtr1: Booting VM...
==> rtr1: Waiting for machine to boot. This may take a few minutes...
    rtr1: SSH address: 127.0.0.1:2201
    rtr1: SSH username: vagrant
    rtr1: SSH auth method: private key

Timed out while waiting for the machine to boot. This means that
Vagrant was unable to communicate with the guest machine within
the configured ("config.vm.boot_timeout" value) time period.

If you look above, you should be able to see the error(s) that
Vagrant had when attempting to connect to the machine. These errors
are usually good hints as to what may be wrong.

If you're using a custom box, make sure that networking is properly
working and you're able to connect to the machine. It is a common
problem that networking isn't setup properly in these boxes.
Verify that authentication configurations are also setup properly,
as well.

If the box appears to be booting properly, you may want to increase
the timeout ("config.vm.boot_timeout") value. 
```
If we type wrong localhost interface name for bridged network there will be option to choose form listed possibilities: 
```bash
==> rtr1: Specific bridge 'enp0s1: Ethernet' not found. You may be asked to specify
==> rtr1: which network to bridge to.
==> rtr1: Available bridged network interfaces:
1) enp0s3
2) docker0
==> rtr1: When choosing an interface, it is usually the one that is
==> rtr1: being used to connect to the internet.
    rtr1: Which interface should the network bridge to?	
    rtr1: Which interface should the network bridge to? 1
```
Now wait for end of csr booting, even 15 minutes. 
Step 5:
  Machines started by vagrant should be vissible in VirtualBox as new instances. 
  Login to csr console via VirtualBox GUI and set basic ssh configuration to allow connections for vagrant and other remotes. 
  ```bash
  conf t
  Router(config)#hostname rtr1
  rtr1(config)#ip domain-name cisco.local
  rtr1(config)#crypto key gen rsa mod 1024
  rtr1(config)#ip ssh ver 2
  rtr1(config)#username vagrant priv 15 secret vagrant
  rtr1(config)#line vty 0 98
  rtr1(config-line)#login local
  rtr1(config-line)#transport input ssh
```
To connect to rtr1 via ssh use command: 
```bash
vagrant ssh rtr1
```
or linux host machine: 
```bash
ssh vagrant@127.0.0.1 -p 2201
```
If needed, add more CSR instances in Vagrantfile: 
```bash
config.vm.define "rtr2" do |rtr2|
    rtr1.vm.box = "cisco-csr1000v-box" 
    rtr1.vm.boot_timeout = 1 
    rtr1.vm.hostname = "rtr2" 
    rtr1.vm.network :forwarded_port, guest: 22, host: 2202, id: "ssh" 
    rtr1.vm.network "public_network",use_dhcp_assigned_default_route: true, bridge: "enp0s3: Ethernet"
    rtr1.vm.network "private_network", ip: "192.168.100.1"
    rtr1.vm.network "private_network",  ip: "192.168.200.1"
    rtr1.vm.provider "virtualbox" do |vb|
      vb.name = "rtr1"
      vb.memory = "3072"
      vb.cpus = 2
    end
  end
  ```
