# bc_linux_network_project
BeCode Linux Module - Linux Project

## Project Context

The local library in your little town has no funding for Windows licenses so the director is considering Linux. Some users are sceptical and ask for a demo. The local IT company where you work is taking up the project and you are in charge of setting up a server and a workstation.
To demonstrate this setup, you will use virtual machines and an internal virtual network (your DHCP must not interfere with the LAN).

You may propose any additional functionality you consider interesting.

## Must Have

Set up the following Linux infrastructure:

1. One server (no GUI) running the following services:
    - DHCP (one scope serving the local internal network)  isc-dhcp-server
    - DNS (resolve internal resources, a redirector is used for external resources) bind
    - HTTP+ mariadb (internal website running GLPI)
    - **Required**
        1. Weekly backup the configuration files for each service into one single compressed archive
        2. The server is remotely manageable (SSH)
    - **Optional**
        1. Backups are placed on a partition located on  separate disk, this partition must be mounted for the backup, then unmounted

2. One workstation running a desktop environment and the following apps:
    - LibreOffice
    - Gimp
    - Mullvad browser
    - **Required** 
        1. This workstation uses automatic addressing
        2. The /home folder is located on a separate partition, same disk 
    - **Optional**
        1. Propose and implement a solution to remotely help a user

# Workstation VM Setup

We'll setup a debian virtual machine with a graphical desktop environment using VirtualBox.

## Steps

### **1. Creating the New VM**

First we need a debian ISO, you can get yours [here](https://www.debian.org/distrib/), I will be using the latest release, debian 12.

Once the ISO downloaded, we can set up the new VM on VirtualBox.

Click on "NEW", fill out the name, select the ISO you just downloaded and check "Skip Unattended Installation".

![](images/vm-creation-1.png)

Allocate at least 2048MB of RAM and 2 CPU cores since we're gonna run a desktop environment on it.

Allocate 30GB of hard drive space or more.

Finish.

---

### **2. Installing Linux on the VM**

A basic linux installation, I will choose the "Install" option, it's more convenient for me than the gaphical install.

Select your language.

Select your location.

Configure the keyboard.

Wait.

Configure the network, I will go with the "workstation" hostname and domain name.

I choose "root" as default password. (security number one priority)

"workstation" for the full name and username, "root" as password. (lol)

Wait, then for the partition use the entire disk, select the disk and <ins>__make sure to separate the /home partition__</ins>, then finish partitioning and write the changes to the disk.

Wait.

Don't scan for the extra installation media, select your mirror country, deb.debian.org as the archive and no proxy.

Participate in the package survey if you wish.

<ins>__For the Software Selection, select Debian desktop environment and select the environment that you like. I will be choosing GNOME and don't forget to tick SSH aswell__</ins> (spacebar to select and deselect)

![](images/vm-creation-2.png)

---

### **3. Setting Up the VM**

Once your machine is turned on, the first thing we have to do is add your user to the sudoers.

```sh
su root
```
```sh
sudo usermod -aG sudo YOUR_USERNAME
```
```sh
exit
```
Restart your VM

-----------------------------------------

We'll start installing the consumer software.

[Libre Office](https://www.libreoffice.org/)
LibreOffice is installed by default on most popular linux distributions, but if it isn't on yours, type
```sh
sudo apt install libreoffice
```

[GIMP](https://www.gimp.org/)
```sh
sudo apt install gimp
```

[MULLVAD Browser](https://mullvad.net/en/download/browser/linux)


```sh
cd Downloads
```
```sh
tar xf mullvad-browser-linux64-[YOUR_VERSION].tar.xf
```
```sh
rm mullvad-browser-linux64-[YOUR_VERSION].tar.xf
```
```sh
mv mullvad-browser ~/
```
```sh
cd
```
```sh
cd mullvad-browser
```
```sh
./start-mullvad-browser-desktop --register-app
```

Now you have mullvad browser registered in your applications

---

We'll also set up the remote help straight away since it's just some more installs.

To be able to remotely connect to a user, we'll use xrdp ([RDP](https://en.wikipedia.org/wiki/Remote_Desktop_Protocol) stands for Remote Desktop Protocol)


```sh
sudo apt update
```
```sh
sudo apt install xrdp
```
```sh
sudo systemctl status xrdp
```

If it's not running, enable it
```sh
sudo systemctl enable --now xrdp
```

Add the xrdp user to the ssl-cert group
```sh
sudo adduser xrdp ssl-cert
```

Restart the xrdp server
```sh
sudo systemctl restart xrdp
```

We can also install Remmina to be able to help a user from another workstation
install remmina

```sh
sudo apt install remmina
```

We can already set up the Uncomplicated Firewall for the future and allow the RDP port

```sh
sudo apt install ufw
```
```sh
sudo ufw allow 3389
```
```sh
sudo ufw enable
```

-----------------------------------------

We'll try to remotely connect to another machine, for that we need to change our VM's network type and set it to Bridged

Shut down your VM, go to Settings > Network and in Adapter 1 switch NAT to Bridged Adapter

![](images/vm-bridged-network.png)

You can now remotely connect to your workstation from another machine on the network

To know your workstation's ip
```sh
ip a
```

It will probably be something like 10.40.X.X

<ins>__You cannot remotely into a machine with an ongoing session with xrdp, either create a guest user or logout__</ins>

-----------------------------------------

### Encoutered problems

<ins>__"Media change: please insert the disc labeled ... in the drive /media/cdrom"__</ins>

It can happen if you fed your machine the CD or DVD version of the ISO, to fix it you need to update your souces list.

```sh
sudo nano /etc/apt/sources.list
```

Delete or comment (#) the CDROM source and paste this:
```
deb http://deb.debian.org/debian bookworm main non-free-firmware
deb-src http://deb.debian.org/debian bookworm main non-free-firmware

deb http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware
deb-src http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware

deb http://deb.debian.org/debian bookworm-updates main non-free-firmware
deb-src http://deb.debian.org/debian bookworm-updates main non-free-firmware
```

And run
```sh
sudo apt update
```

-----------------------------------------


# server vm setup - todo

same setup as the workstation except during the step where you choose the desktop environment (GNOME, Xfce, etc..), deselect every environment and select webserver and SSH

-----------------------------------------

bridged + ssh

-----------------------------------------
INSTALL SUDO AND ADD USER TO SUDOERS

>su root
>apt install sudo
>usermod -aG sudo [your_username]
>exit

to test sudo and privilege, try
>sudo apt update

-----------------------------------------

>sudo apt install ufw
>sudo ufw enable
>sudo ufw allow SSH,80,443,53

(SSH for ssh, 80 for HTTP, 443 for HTTPS and 53 for DOMAIN)

-----------------------------------------

SET STATIC IP TO SERVER

>sudo apt install network-manager

>sudo nano /etc/network/interfaces

comment out

allow-hotplug enp0s3
iface enp0s3 inet dhcp

---

BE CAUTIOUS TO REPLACE THE X's AND PROVIDE AN IP FOR YOUR SERVER

>nmcli con mod "Wired connection 1" ipv4.addresses "10.40.X.X/16" ipv4.gateway "10.40.0.1" ipv4.dns "" ipv4.dns-search "" ipv4.method "manual"

you now have a server with a static ip

-----------------------------------------

GLPI (http + mariadb)
https://github.com/jr0w3/GLPI_install_script

>su root
>wget https://raw.githubusercontent.com/jr0w3/GLPI_install_script/main/glpi-install.sh && bash glpi-install.sh

follow the instructions

we are supposed to restart apache2 but it wont work, so we run this

>sudo ln -s /etc/apache2/mods-available/rewrite.load /etc/apache2/mods-enabled/rewrite.load
>sudo systemctl restart apache2
>sudo systemctl enable apache2

so now you can access your glpi server by opening a browser and typing in your machine's ip, to find it, type

>ip a

credentials for glpi are glpi:glpi

-----------------------------------------

DNS INSTALL

follow [this tutorial](https://computingforgeeks.com/configure-master-bind-dns-server-on-debian/)

YOU CAN USE NANO INSTEAD OF VIM

IN STEP 3 YOU CAN OMIT THE MAIL EXCHANGER CONFIG FOR THE FORWARD DB AND THE PTR RECORD IP FOR THE REVERSE DB

IN STEP 4 WHEN RESTARTING, ENABLING AND CHECKING YOUR DNS, REPLACE 'bind9' with 'named'

IN STEP 5 WHEN WRITING IN THE RESOLV.CONF, PUT THE LINE WITH NAMESERVER [SERVER_IP] AT THE TOP OF THE FILE IF SOME LINES ARE ALREADY PRESENT

IGNORE STEP 6

-----------------------------------------

INSTALLING AND SETTING UP DHCP SERVER

>sudo apt install isc-dhcp-server

>sudo nano /etc/default/isc-dhcp-server

replace the last 2 lines:

INTERFACESv4="enp0s3"
#INTERFACESv6=""

>sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.old
>sudo nano /etc/dhcp/dhcpd.conf

paste:

default-lease-time 600;
max-lease-time 7200;

ddns-update-style none;

authoritative;

option subnet-mask 255.255.0.0;
option broadcast-address 10.40.255.255;
option domain-name-servers [YOUR_SERVER_IP];
option domain-name "[YOUR_DOMAIN_NAME]";
option routers 10.40.0.1;

subnet 10.40.0.0 netmask 255.255.0.0 {
        range 10.40.X.X 10.40.X.X;
}

FOR THE SUBNET MAST RANGE I SUGGEST YOU CHOOSE A SMALL RANGE, i.e:

subnet 10.40.0.0 netmask 255.255.0.0 {
        range 10.40.5.100 10.40.5.120;
}

DO NOT INCLUDE YOUR DHCP'S SERVER IP, SO CHOOSE A RANGE ABOVE OR BLOW YOUR SERVERS IP

save, restart both VM's

to verify that it's working proprely, **on your workstation**, run
>sudo dhclient -v

you should see a line like this
DHCPOFFER of [YOUR_NEW_IP] from [YOUR_SERVER_IP]

THE FINAL STEP TO SEE IF DHCP AND DNS ARE WORKING TOGETHER, OPEN A BROWSER AND GO TO [YOUR_DOMAIN_NAME] THAT YOU SET UP DURING THE DNS CONFIGURATION. IF EVERYTHING IS SET UP CORRECTLY, YOU SHOULD ACCESS YOUR GLPI INTERFACE.

ie, i chose wares.bobby.local so thats the url i'm gonna type in the browser (do not forget the subdomain)

-----------------------------------------

BACKUPS + OPTIONAL

    Backups are placed on a partition located on separate disk, this partition must be mounted for the backup, then unmounted


first you need to create a new volume for your vm

1. close your vm
1. on virtualbox, while your vm is selected, go to settings
1. go to storage
1. click on the little hard drive on the left of Controller:SATA
1. create on the top left
1. VDI
1. next and then allocate the space you want (at least 5gb)
1. select your new volume and click on choose


you now have a new volume attached to your vm

to check your disks on your vm, type
>lsblk
if you see a /dev/sdb then your additional drive exists

so we'll start by formatting it to ext4
>sudo mkfs.ext4 /dev/sdb

we'll create a backup dir in /mnt
>sudo mkdir /mnt/conf_backups

then we'll create a script to mount our disk and backup our conf files before unmounting it

>su root
>cd
>mkdir scripts
>nano /root/scripts/conf_backup.sh

sudo mount /dev/sdb /mnt/conf_backups/
sudo mkdir /tmp/$(date +%d-%b-%Y)
sudo cp -r /etc/bind /etc/dhcp /etc/apache2 /etc/mysql /etc/php /etc/resolv.conf /tmp/$(date +%d-%b-%Y)/
sudo tar -zcvf /mnt/conf_backups/$(date +%d-%b-%Y).tar.gz /tmp/$(date +%d-%b-%Y)/
sudo rm -rf /tmp/$(date +%d-%b-%Y)
sudo umount /dev/sdb

then we create a cronjob to launch the script every week on sunday at 9pm
>sudo crontab -u root -e
>00 21 * * 7 /root/scripts/conf_backup.sh

IF THE SCRIPT ISNT EXECUTABLE BY ROOT (CREATED BY ROOT), MAKE SURE TO MAKE YOUR SCRIPT EXECUTABLE FOR ROOT
>sudo chmod 770 [PATH_TO_YOUR_SCRIPT]

-----------------------------------------

USEFUL LINKS

rsyslog
nmap

-----------------------------------------

TROUBLES ENCOUNTERED

problem with debian 11 under gnome we can use this http://c-nergy.be/blog/?p=18918

>wget https://c-nergy.be/downloads/xRDP/xrdp-installer-1.4.7.zip
>cd ~/Downloads
>unzip xrdp-installer-1.4.7.zip
>sudo chmod +x xrdp-installer-1.4.7.sh
>sh xrdp-installer-1.4.7.sh -c -l

-----------------------------------------
