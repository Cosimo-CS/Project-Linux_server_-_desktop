# Project Linux server and desktop #
1st linux project - Kamkar 04.24

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

----------------------------------------------------------------------------------------------

In this project you can find a setup with a desktop (Ubuntu Linux) and a server (Linux server) within a virtualized environment on Oracle virtual box

In this repository you can find the configuration of two virtual machines: 

- Server: Ubuntu 22.04.4 TLS
- Desktop: Ubuntu server 22.04

The server is installed without GUI.

![alt text](assets/bind9test.png)

Let's discover the differents configurations made for this project:

- Server configuration --> linux-server-config.md
- Desktop configuration --> linux-desktop-config.md


# Server Ubuntu VM:

Settings:
- 4096 Mo of RAM
- 3 CPU cores
- 20 GB of storage
- Adaptater 1 : Bridged network adapter
- Adaptater 2 : Private Host 
- Network manager CFG (Virtual box) : ipv4 : 192.168.52.2 DHCP: Disabled
- Ubuntu 22.04 LTS Server Edition
- Hostname: `ubuntu-server`

### __1. Creating the New VM__

First of all you need to download your .iso file. 

You can get one in the following link: [ubuntu-22.04-iso](https://ubuntu.com/download/server)

Once the ISO downloaded, we can set up the new VM on VirtualBox.

Click on New button, fill out the name, select the ISO you just downloaded.

![alt-text](assets/iso-vm.png)

Press "Next" and there you can set up your username with password and the hostname of the server.

Allocate 4096MB of RAM;
Allocate 20GB of hard drive space or more. (Depending on the space in your host computer)

Finish.

------------------------------------------------------------------------------------

### __2. Installing Linux on the VM__

Select your language: I choosed English by default.

Configure the keyboard: 
- Layout: Belgian
- Variant: Belgian

Done.

Choose the base for the installation:
- Ubuntu server

Done.

## Network connections

Don't touch nothing and just press done.

Same for "Configure proxy", "Configure ubuntu archive mirror" just press done, we don't need to set up one for this project.

## Guided storage configuration

You can leave it by default also:
- Use an entire disk
- Set up this disk as an LVM group

Done.

## Storage configuration

I left the configuration by default also.

Done and you can confirm destructive action.

## Profile setup

- Your name: Totti K
- Your servers name: ubuntu-server
- Pick a username: totti
- choose a password: ******
- Confirm it.

Done.

## SSH setup

I didn't Installed open SSH server we will configure it later, same for Feature Server Snaps.

Congrats! You can take a coffee during the installation of your server. x-)

When it's done you can press on "Reboot now"

### __3. Setting Up the VM__

Now let's get down to business and start configuring the server!

As you can see, it's quite annoying to 'not be able' to do what you want because you're only faced with a terminal.

So I wanted to simplify my life and set up an SSH connection directly with my desktop (which I'd already installed upstream).

But first let's do some basic configuration, which will be necessary for the rest of the operation.

## Steps

### __1. Install sudo and add user to sudoers__

Install sudo and add your user to the sudoers group:
```sh
su root
```
```sh
apt install sudo
```
```sh
usermod -aG sudo YOUR_USERNAME
```
```sh
exit
```

Test sudo and privilege:
```sh
sudo apt update
```

I've also added few nice commands to get information from your new fresh server. They are in the commands file.
