# bc_linux_project
BeCode Linux Module - Linux Project

## Project context

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

# workstation vm setup

## introduction
setting up a debian graphical workstation vm for the linux project

## steps

### creating the new vm

### installing linux on the vm

### setting up the vm

IF YOUR USER IS NOT IN THE SUDOERS GROUP, ADD IT

>su root
>sudo usermod -aG sudo [your username]
>exit

restart your vm

-----------------------------------------

IF YOU HAVE THE "Media change: please insert the disc labeled ... in the drive /media/cdrom"

it can happen if you fed your machine the CD or DVD version of the iso

then you need to update your sources list

>sudo nano /etc/apt/sources.list

delete or comment (#) the CDROM source and paste this:

deb http://deb.debian.org/debian bookworm main non-free-firmware
deb-src http://deb.debian.org/debian bookworm main non-free-firmware

deb http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware
deb-src http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware

deb http://deb.debian.org/debian bookworm-updates main non-free-firmware
deb-src http://deb.debian.org/debian bookworm-updates main non-free-firmware

and run 
>sudo apt update

-----------------------------------------

INSTALL CONSUMER PROGRAMS

[Libre Office](https://www.libreoffice.org/)
LibreOffice is installed by default on most popular linux distributions, but if it isn't on yours, type
>sudo apt install libreoffice

[GIMP](https://www.gimp.org/)
>sudo apt install gimp

[MULLVAD Browser](https://mullvad.net/en/download/browser/linux)

>cd Downloads
>tar xf mullvad-browser-linux64-[YOUR_VERSION].tar.xf
>rm mullvad-browser-linux64-[YOUR_VERSION].tar.xf
>mv mullvad-browser ~/
>cd
>cd mullvad-browser
>./start-mullvad-browser-desktop --register-app

Now you have mullvad browser registered in your applications

-----------------------------------------

REMOTELY HELP A USER

to help from server to client
install xrdp

>sudo apt update
>sudo apt install xrdp
>sudo systemctl status xrdp

if it's not running, enable it
>sudo systemctl enable --now xrdp

add xrdp user to ssl-cert group
>sudo adduser xrdp ssl-cert

restart xrdp server
>sudo systemctl restart xrdp



to help from client to client
install remmina

>sudo apt install remmina

-----------------------------------------

INSTALL UNCOMPLICATED FIREWALL

install ufw

>sudo apt install ufw
>sudo ufw allow 3389
>sudo ufw enable

-----------------------------------------

CONNECTING TO A MACHINE

enabling bridged network type in virtualbox's vm settings

you can then remotely connect to your workstation from another machine on the network

to know your workstation's ip
>ip a

it will probably look like 10.40.X.X

YOU CANNOT LOGIN INTO AN ONGOING SESSION WITH XRPD, EITHER CREATE A GUEST USER OR LOGOUT AND TRY AGAIN

-----------------------------------------
-----------------------------------------
-----------------------------------------

# server vm setup

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
