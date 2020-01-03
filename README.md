# vagrant_csr1000v
Spinning csr1000v router using vagrant with virtualbox as provisioner. 

Whate we need:
  - csr1000v iso image.
  - installed vagrant
  - installed virtual box

Step 1:
  In virtualbox gui click new machine.
  Set Name: csr1000v (for example). 
  Set Type: Linux. 
  Set Version: Other Linux (64-bit). 
  Choose Create virtual hard disk now > VDI (VIrtualBox Disk Image) > Fixed size > Default disk name and location > Size 8GB. 
  Right click on created csr machine > Settings... > Serial Ports > Thick Enable Serial Port > Port Mode: Disconnected. 
  Start your machine point to csr iso image if not mounted before spinning it up in Settings > Storage. 
  Shut if down and be sure the iso image is unmounted form drive. 
Step 2: 
Make new directory for vagrant project for example: Vagrant in your Home directory for example and Vagrant/Boxes for package. 
Type in shell: 
  vagrant package --base csr1000v --output /home/adrian/Vagrant/Boxes/
    ==> csr1000v: Exporting VM...
    ==> csr1000v: Compressing package to: /home/adrian/Vagrant/boxes/cisco-csr1000v
      (csr1000v is your machine name created in VirtualBox via GUI) 
      
  ll -h /home/adrian/Vagrant/boxes/ 
    total 1.1G
    drwxrwxr-x 2 adrian adrian 4.0K Jan  3 20:22 ./
    drwxrwxr-x 4 adrian adrian 4.0K Jan  3 22:45 ../
    -rw-rw-r-- 1 adrian adrian 1.1G Jan  3 20:24 cisco-csr1000v 
      (vagrant package for csr should be created) 
      
  vagrant box add /home/adrian/Vagrant/boxes/cisco-csr1000v --name cisco-csr1000v-box
    ==> box: Box file was not detected as metadata. Adding it directly...
    ==> box: Adding box 'cisco-csr1000b-box' (v0) for provider: 
        box: Unpacking necessary files from: file:///home/adrian/Vagrant/boxes/cisco-csr1000v
    ==> box: Successfully added box 'cisco-csr1000b-box' (v0) for 'virtualbox'!
  vagrant box list
    cisco-csr1000v-box (virtualbox, 0)
      (vagrant box should be created)
  Step 3: 
    Now we need to create "Vagrantfile" in our project directory by creating it in any text editor or by typing in shell:
      vagrant init
    then edit Vagrantfile in nano for example. 
    
Vagrant.configure("2") do |config|
  
  config.vm.define "rtr1" do |rtr1|
      rtr1.vm.box = "cisco-csr1000v-box"
      rtr1.vm.boot_timeout = 1
      rtr1.vm.hostname = "rtr1"
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

