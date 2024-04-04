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

Let's discover differents configurations made for this project:

Server configuration --> linux-server.config
Desktop configuration --> linux-desktop.config

Enjoy! 





