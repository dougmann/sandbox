# Xilinx Petalinux Installation
First you need to install the Vivado ML Edition - 2022.1  Full Product Installation:   
https://www.xilinx.com/support/download.html


```
sudo apt update
sudo apt upgrade
sudo apt install libncurses5
sudo apt install libtinfo5
sudo apt install libncurses5-dev libncursesw5-dev
```

```
sudo dpkg --add-architecture i386
```
```
sudo dpkg-reconfigure dash
```


sudo apt-get install iproute2 gawk python3 python build-essential gcc git make net-tools libncurses5-dev tftpd zlib1g-dev libssl-dev flex bison libselinux1 gnupg wget git-core diffstat chrpath socat xterm autoconf libtool tar unzip texinfo zlib1g-dev gcc-multilib automake zlib1g:i386 screen pax gzip cpio python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 libegl1-mesa libsdl1.2-dev pylint3



Download the Xilinx Unified Installer:  
Xilinx Unified Installer 2022.1: Linux Self Extracting Web Installer (BIN - 266.73 MB)

Make sure the installer has execute priviledges and run the installer:  
```
cd ~/Downloads
sudo chmod +x Xilinx_Unified_2022.1_0420_0327_Lin64.bin
sudo ./Xilinx_Unified_2022.1_0420_0327_Lin64.bin
```

![installer_options](https://user-images.githubusercontent.com/11302627/170825588-b4eb45b3-2c86-4516-9552-1c978ea6f386.png)

Make sure to enable support for Zynq-7000 and Ultrascale  
![vitis_options](https://user-images.githubusercontent.com/11302627/170825678-70fc97a5-eecf-4a9b-902a-2b841ded1509.png)

Click "Next" to use defaults and click "Install"  


Create TFTP Server

PetaLinux also requires a TFTP server service to support TFTP booting on a target system. In the /etc/xinetd.d/ directory create a TFTP service file:
```
sudo gedit /etc/xinetd.d/tftp
```
And configure it as such:
```
service tftp 
    {
    protocol = udp 
    port = 69 
    socket_type = dgram 
    wait = yes 
    user = nobody 
    server = /usr/sbin/in.tftpd 
    server_args = /tftpboot 
    disable = no
    }
   ``` 
    
    Then create the directory for the TFTP service to pull files from during the target boot process such as the boot image file (BOOT.bin), kernel, device tree, etc. Give the directory the appropriate permissions and give ownership to the same user specified in the TFTP service.
    
    
```
sudo mkdir /tftpboot
sudo chmod -R 777 /tftpboot
sudo chown -R nobody /tftpboot
```

Stop and restart the host machine's extended internet services for these changes to take effect.

```
sudo /etc/init.d/xinetd stop
sudo /etc/init.d/xinetd start
```

Add Your User to Dialout Group

Add your local user to the dial out group if you haven't already so Vivado and Vitis can access the computer's USB ports for serial communication with FPGA targets.
```
sudo adduser $USER dialout
```
At this point, I recommend rebooting your machine to make sure everything is implemented.



Run Vitis Installer

While the downloads website has separate tabs for Vivado and Vitis, the Vitis installer also includes the option to install Vivado. The Vivado installer however only contains the option to install Vivado and if you try to then run the Vitis installer later, it will force you to install it in a different location. This can get messy very quickly, so I highly recommend only using the Vitis installer because it also contains the option to only install Vivado. The difference is that you'll be able to run the installer again later and install Vitis in the same location with Vivado.


If using the single file all-OS installer, extract the installer folder from the downloaded compressed package and run the setup installer GUI script:
```
sudo ./Dowloads/Xilinx_Unified_2021.2_1021_0703/xsetup
```
If using the Linux self-extracting web installer, give it the appropriate permissions to make it executable and run it:
```
sudo chmod 777 ./Downloads/Xilinx_Unified_2021.2_1021_0703_Lin64.bin
sudo ./Downloads/Xilinx_Unified_2021.2_1021_0703_Lin64.bin
```
The GUI by default will install Vitis and Vivado to /tools/Xilinx/. You can change this if desired, but I highly recommend leaving it in this location.

Test installation by launching Vivado and Vitis. First, launch Vivado:
```
source /tools/Xilinx/Vivado/2022.1/settings64.sh
vivado
```
Close Vivado or open a new terminal window and launch Vitis:
```
source /tools/Xilinx/Vitis/2022.1/settings64.sh
vitis
```
Run PetaLinux Installer

You can install the PetaLinux mostly wherever you prefer, but I like to keep all of the Xilinx tools in the same place so I create a PetaLinux directory in the same directory Vivado and Vitis installed to following the version format as well:
```
sudo mkdir -p /tools/Xilinx/PetaLinux/2022.1/
```
Give the directory 755 permissions (making the folder globally read-execute):
```
sudo chmod -R 755 /tools/Xilinx/PetaLinux/2022.1/
```
Give the PetaLinux installer 777 permissions:
```
sudo chmod 777 ./Downloads/petalinux-v2022.1-final-installer.run
```
Change ownership of the directory you’re installing PetaLinux in to the user:
```
sudo chown -R <user>:<user> /tools/Xilinx/PetaLinux/2022.1/
```
Run the PetaLinux installer:
```
~/Downloads/petalinux-v2022.1-final-installer.run /tools/Xilinx/PetaLinux/2022.1/
```
Then test the installation by turning Webtalk on/off:
```
source /tools/Xilinx/PetaLinux/2022.1/settings.sh
petalinux-util --webtalk off
petalinux-util --webtalk on
```
Notes

It's worth noting that a PetaLinux project can not be built offline without access to the internet unless specifically configured to do so. This involves downloading the proper repositories locally to your machine and pointing the PetaLinux project to it, which I'll cover in a different post. This is something that you'll have to do to each PetaLinux project individually that you want to be able to run a build on offline.

Vivado also requires a network connection to download/install new development board preset files. So you'll want to make sure you can access the internet at least the first few times you create a new project.


